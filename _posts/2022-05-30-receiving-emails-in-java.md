---
layout: post
title: Receiving mails in Java with IMAP or POP3
author: jens_knipper
date: '2022-05-30 01:00:00'
description: I was recently in need to write some small demo project which was receiving and processing mails. There is a lot of documentation for receiving mails, but gathering information about the sending part is less easy.
categories: Java, Mail, IMAP, POP3
---
This example shows you how to receive mails in Java either using the IMAP or the POP3 protocol.
The SSL encrypted variants IMAPS and POP3S are also supported. 
Received mails will be set to read, which means that you will only receive the latest ones.
I will also give some hints about how to handle incoming mails concerning application security.
 Remember to alwalys sanitize user inputs. 

## Receiving mails

You need to use to the following import statement `import javax.mail.*;` to be able to connect mail servers.
The constructor of the client is straightforward. 
It is simply used to pass and store some values into fields, which are then used later on.
We are interested in the protocol, host, port user and password.
Valid values for the protocol are imap, imaps, pop3 and pop3s.

{% highlight java %}
public final class MailReceiveClient {
    private final String protocol;
    private final String host;
    private final String port;
    private final String user;
    private final String password;

    public MailReceiveClient(String protocol, String host, String port, String user, String password) {
        this.protocol = protocol;
        this.host = host;
        this.port = port;
        this.user = user;
        this.password = password;
    }
{% endhighlight %}

We use the following code sample to connect to the mail server.
First we have to create new properties and add values with the defined protocol, host and port.
We use these properties to create a Session.
With this session we can create a Store and connect to the server using the given username and password.  
Afterwards we get the inbox folder and open it in read write mode. 
Inbox usually is the default folder.
If you want to get a different one, just change this string.
The folder is opened also in write mode to mark mails as read.
We pass this folder into the `getNewMails` method to receive the latest mails.  
There is some wrapping of exceptions into runtime exceptions in the end.
Folder and Store also have to be disconnected in the end.

{% highlight java %}
    public List<Mail> receive() {
        Store emailStore = null;
        Folder emailFolder = null;

        Properties properties = new Properties();
        properties.put("mail.store.protocol", protocol);
        properties.put("mail." + protocol + ".host", host);
        properties.put("mail." + protocol + ".port", port);
        Session emailSession = Session.getInstance(properties);

        try {
            emailStore = emailSession.getStore();
            emailStore.connect(user, password);

            emailFolder = emailStore.getFolder("INBOX");
            emailFolder.open(Folder.READ_WRITE);

            return getNewMails(emailFolder);

        } catch (MessagingException e) {
            throw new RuntimeException(e);
        } finally {
            try {
                if (emailFolder != null && emailFolder.isOpen()) {
                    emailFolder.close(false);
                }
                if (emailStore != null && emailStore.isConnected()) {
                    emailStore.close();
                }
            } catch (MessagingException e) {
                throw new RuntimeException(e);
            }
        }
    }
{% endhighlight %}

The `getNewMails` method gets all the messages in the folder.
An if condition is used to only process unseen messages.
Afterwards the message is set to seen.
Keep in mind that no data is beeing deleted.
When processing big volumes of mails your folder gets bigger quite fast.
You might want to delete messages to keep the mail processing short.
The code uses a for loop to avoid race conditions.
In case there are multiple mail clients connected mails are not processed multiple times.
A mapping to an internal class is needed, because an exception is thrown as soon as you try to access a message who's session has been closed. Another reason is the sanitizing of user inputs. You can read more about this in the next chapter.

{% highlight java %}
    private List<Mail> getNewMails(Folder emailFolder) throws MessagingException {
        List<Mail> mails = new ArrayList<>();
        for (Message message : emailFolder.getMessages()) {
            if (!message.getFlags().contains(Flags.Flag.SEEN)) {
                message.setFlags(new Flags(Flags.Flag.SEEN), true);
                Mail mail = MailMapper.map(message, user);
                mails.add(mail);
            }
        }
        return mails;
    }
}
{% endhighlight %}

The class can also be found on [GitHub](https://github.com/JensKnipper/greenmail-example/blob/main/src/main/java/de/jensknipper/greenmailexample/control/mail/receive/MailReceiveClient.java).

The error handling is just an example how it could be done. Depending on your needs you might want to use a more sophisticated one.

In the end a List of mails is returned. The mail class is just a simple data class containing information we extract from the messages on the mail server. For my minimal example I was only interested in the subject, the content, the sender and the recipient (even though there can be more than one recipient). You can extract even more informations from the messages. The documentation of the class can be found [here](https://javaee.github.io/javamail/docs/api/javax/mail/Message.html).

{% highlight java %}
public final class Mail {
    private final String subject;
    private final String content;
    private final String from;
    private final String recipient;

    public Mail(String subject, String content, String from, String recipient) {
        this.subject = subject;
        this.content = content;
        this.from = from;
        this.recipient = recipient;
    }

    public String getSubject() {
        return subject;
    }

    public String getContent() {
        return content;
    }

    public String getFrom() {
        return from;
    }

    public String getRecipient() {
        return recipient;
    }
}
{% endhighlight %}

You can read more about the mapping in the next chapter.

## Sanitizing received Mails

When you are dealing with user input (you could also count mails to this) you should always worry about code injections. The best way to prevent these is sanitizing the inputs you receive. You can obviously do this with mails as well.
OWASP provides a nice utility called [Encoder](https://owasp.org/www-project-java-encoder/) to do that.
I use a simple helper class with static methods to do the mapping. All the attributes received by mail will be encoded for displaying it in HTML. The encoder also brings methods for inserting your mail attributes into e.g. JavaScript or CSS.
We simply extract the things we are interested in from the message and sanitize the input using the encoder before creating a new mail object with the attributes.

{% highlight java %}
import org.owasp.encoder.Encode;

import javax.mail.Address;
import javax.mail.Message;
import javax.mail.MessagingException;
import java.io.IOException;
import java.util.Arrays;

public final class MailMapper {

    private MailMapper() {}

    public static Mail map(Message message, String user) {
        String subject = Encode.forHtml(getSubject(message));
        String content = Encode.forHtml(getContent(message));
        String from = Encode.forHtml(getFrom(message));
        String recipient = Encode.forHtml(getRecipient(message, user));
        return new Mail(subject, content, from, recipient);
    }

    private static String getSubject(Message message) {
        try {
            return message.getSubject();
        } catch (MessagingException e) {
            throw new RuntimeException(e);
        }
    }

    private static String getContent(Message message) {
        try {
            Object content = message.getContent();
            if (content == null) {
                return null;
            }
            return content.toString();
        } catch (IOException | MessagingException e) {
            throw new RuntimeException(e);
        }
    }

    private static String getFrom(Message message) {
        try {
            Address[] from = message.getFrom();
            if (from.length == 0 || from[0] == null) {
                return null;
            }
            return from[0].toString();
        } catch (MessagingException e) {
            throw new RuntimeException(e);
        }
    }

    private static String getRecipient(Message message, String user) {
        try {
            return Arrays.stream(message.getAllRecipients())
                    .map(Address::toString)
                    .filter(it -> it.startsWith(user))
                    .findFirst()
                    .orElse(user);
        } catch (MessagingException e) {
            throw new RuntimeException(e);
        }
    }
}
{% endhighlight %}

The class can also be found on [GitHub](https://github.com/JensKnipper/greenmail-example/blob/main/src/main/java/de/jensknipper/greenmailexample/control/mail/mapper/MailMapper.java).


## Usage

To get started simply fill in the constructor's parameters to connect to your mail server. 
An example for IMAP might look something like this:  
`new MailReceiveClient("imap", "localhost", "143", "user", "password");`

And for POP3 like this:  
`new MailReceiveClient("pop3", "localhost", "110", "user", "password");`

You could also use Spring to create a bean using the `Component` annotation and set the constructor's parameters via properties using the `Value` annotation. This is the way I am doing it in my [demo project](https://github.com/JensKnipper/greenmail-example).

If you do not know how to easily start up a local (mock) mailserver you can use GreenMail to do that. I have written an article how to [integrate it into you local environment](../greenmail-mock-mail-server-dev-setup/).

## Testing

There is quite some logic in this piece of code apart from the connection to the server. Things that should get tested. I recently held a [presentation at a meetup about testing mails](../openvalue-meetup-greenmail-talk). You can use this resource to get started. Along to the repo I wrote a test for this class, which you can check out on [GitHub](https://github.com/JensKnipper/greenmail-example/blob/main/src/test/java/de/jensknipper/greenmailexample/control/mail/receive/MailReceiveClientTest.java). There might also be a follow up article in the future about testing emails in Java.

As always full source code is available on [GitHub](https://github.com/JensKnipper/greenmail-example/blob/main/src/main/java/de/jensknipper/greenmailexample/control/mail/receive/MailReceiveClient.java).