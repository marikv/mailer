# Mailer 

Mailer is a simple package based on SwiftMailer which allow sending messages with different drivers.

### Instalation
You can use the `composer` package manager to install. Either run:

    $ php composer.phar require marikv/mailer "dev-master"

or add:

    "marikv/mailer": "dev-master"

to your composer.json file

# Configuration

If you use DotEnv package all you have to do is to populate your ***.env*** file.

```env
MAIL_DRIVER=smtp // Allow to select transport
MAIL_HOST=smtp.gmail.com
MAIL_PORT=465
MAIL_USERNAME=your_gmail@gmail.com
MAIL_PASSWORD=your_password
MAIL_ENCRYPTION=ssl
```

and start using 

```php
$mailer = (new Mailer(
    new Mailer\Transport
))->alwaysFrom('your_email_always_from@gmail.com', 'Your name')
    ->alwaysReplyTo('reply_to@gmailcom', 'Reply name');
```

If you use other storage repository you can create new Mailer instance and populate with config data from array

```php
$config = array(
    'driver' => 'smtp',
    'host' => 'smtp.gmail.com',
    'port' => 465,
    'username' => 'you_email@gmail.com',
    'password' => 'your_passwd',
    'encryption' => 'ssl'
);

$mailer = (new Mailer\Mailer(
    new Mailer\Transport( $config )
))->alwaysFrom('your_email_always_from@gmail.com', 'Your name')
    ->alwaysReplyTo('reply_to@gmailcom', 'Reply name');
```

If you want add custom transport all you have to do is to create new transport class which implement ***TransportAble***

```php

class ArrayTransport extends Transport
    implements TransportAble {
    
    /**
     * {@inheritdoc}
     */
    public function send(Swift_Mime_Message $message, &$failedRecipients = null) {
        $this->beforeSendPerformed($message);
        
        // Send your message
    
        return $this->numberOfRecipients($message);
    }
}
```

And register driver to Transport class.

```php
$tranport = (new Mailer\Transport([
    'driver' => 'my_driver'
]))->extend('my_driver', function($transport) {
    return new ArrayTransport()
});
    
$mailer = (new Mailer($tranport))
->alwaysFrom('your_email_always_from@gmail.com', 'Your name')
    ->alwaysReplyTo('reply_to@gmailcom', 'Reply name');

```
    
# Usage

You can use directly ***Mailer*** and send messages by using ***::to('to_email@gmail.com')*** method

```php

$mailer->to('to_email@gmail.com', 'Subject')
    ->send('This is a test message');

$mailer->to('to_email@gmail.com', 'Subject')->send('Message body', null, function (\Mailer\Message $message) {
    return $message->setFrom('other_from_email@gmail.com');
});
```

or you can create ***Mailable*** classes which require ***build*** method

```php
class NewPayment extends \Mailer\Mail
    implements \Mailer\Mailable {

    public function build() {
        $this->text('Body message');
    }
}

$mailer->to(array(
    'to_email@gmail.com'
))->send( new NewPayment('Subject message') );
```

You can change driver using ***with*** method:

```php
$mailer->to('to_email@gmail.com', 'Subject')
    ->with('sendmail')
    ->send('This is a test message');
```