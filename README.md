[![Continuous Integration](https://github.com/lm-commons/LmcMail/actions/workflows/continuous-integration.yml/badge.svg)](https://github.com/lm-commons/LmcMail/actions/workflows/continuous-integration.yml)
[![Latest Stable Version](https://poser.pugx.org/lm-commons/lmc-mail/v/stable)](https://packagist.org/packages/lm-commons/lmc-mail)
[![Total Downloads](http://poser.pugx.org/lm-commons/lmc-mail/downloads)](https://packagist.org/packages/lm-commons/lmc-mail)
[![License](http://poser.pugx.org/lm-commons/lmc-mail/license)](https://packagist.org/packages/lm-commons/lmc-mail)
[![PHP Version Require](http://poser.pugx.org/lm-commons/lmc-mail/require/php)](https://packagist.org/packages/lm-commons/lmc-mail)
# LmcMail

An email service module that provides the ability to use the View renderer of a Laminas MVC application
and the installed View Helper plugins to render HTML emails.


## Requirements

- PHP 8.1 or higher
- Laminas MVC

## Installation

Install the module:

````shell
$ composer require lm-commons/lmc-mail
````

Composer will inject the module into the modules configuration, or you can add it manually to the `modules.config.php` or
`application.config.php`.

Customize the module by copying and renaming the sample configuration file `lm-commons/lmc-mail/config/lmcmail.local.php.dist` to the application's 
`config/autoload`.

## Configuration

LmcMail supports with the Laminas SMTP Mail Transport or the Laminas File Mail Transport and this is configured by the `transport` config key in the `lmcmail.local.php` file:

````php
<?php
return [
    'lmc_mail' => [
        'from' => [
            'email' => 'user@example.com',
            'name' => 'User',
        ],

        // For SMTP
        'transport' => [
            'type' => 'smtp',
            'options' => [
                'host' => 'example.com',
                'connection_class' => 'plain',
                'connection_config' => [
                    'ssl' => 'tls',
                    'username' => 'user@example.com',
                    'password' => 'somepassword',
                ],
                'port' => 587,
            ],
        ]    
         // OR
            
        'transport' => [
            'type' => 'file',
            'options' => [
                'path' => '/path/to/email/folder',
            ],
        ],
    ],
];
````
In a development environment, it is typical to use a File Mail Transport.  In a production environment, an SMTP Mail Transport will more likely be used.

The `'transport'` configuration must comply with the `Laminas\Mail\Transport\Factory\Factory::create` method.

The `'from'` configuration defines a default *from* address.  The *from* address can also be specified at message creation.

## Usage

The Mail service can be retrieved from the service manager:

````php
$messageService = $serviceManager->get(LmcMail\Service\MessageService::class);
````

Basic example to send an HTML email:

````php
$viewModel = new \Laminas\View\Model\ViewModel();
$viewModel->setTemplate('mail/html');
$message = $messageService->createHtmlMessage(
    ['email' => 'john@example.com', 'name' => 'John'], //from 
    ['email' => 'jane@example.com', 'name' => 'Jane'] //to
    "This is the subject line", //subject
    $viewModel); // View model

$messageService->send($message);
````

The `'mail/html'` template must exist in the application's view template map. The HTML mail renderer will use
a layout template aliased as `'mail/layout'` in the view template map. This is defined in the `module.config.php` file.

### Available methods

#### createHtmlMessage

````php
 /**
  * Create an HTML message
  * @param string|Address|AddressInterface|array|AddressList|Traversable $from
  * @param string|Address|AddressInterface|array|AddressList|Traversable $to
  * @param string $subject
  * @param string|ModelInterface $nameOrModel
  * @return Message
  */
createHtmlMessage(string|Address|AddressInterface|array|AddressList|Traversable $from, 
                  string|Address|AddressInterface|array|AddressList|Traversable $to, 
                  string $subject, 
                  string|ModelInterface $nameOrModel): \Laminas\Mime\Message::class
````
If `$nameorModel` is a string, it must correspond to the view template to use. 


#### createTextMessage
````php
/**
 * Create a text message
 * @param string|Address|AddressInterface|array|AddressList|Traversable $from
 * @param string|Address|AddressInterface|array|AddressList|Traversable $to
 * @param string $subject
 * @param string|ModelInterface $nameOrModel
 * @return Message
 */
createTextMessage(string|Address|AddressInterface|array|AddressList|Traversable $from, 
                  string|Address|AddressInterface|array|AddressList|Traversable $to, 
                  string $subject, 
                  ModelInterface $nameOrModel): \Laminas\Mail\Message::class
````
If `$nameorModel` is a string, it must correspond to the view template to use.

#### send
````php
/**
 * Send the message
 * @param Message $message
 */
send(Message $message): void
````
where `$message` can be any object of type `\Laminas\Mail\Message` not necessarily one created by the above methods.

### Advanced Customizations

LmcMail can be customized to the applications needs.

#### Using view templates

LmcMail uses nested view models to render the body of HTML messages. 

In a similar fashion to the view model structure of the Laminas MVC Skeleton,
the body is rendered using a layout view model to which the view model parameter (`$nameOrModel`) to the `createHtmlMessage` method is added a child.
The rendered output of the `$nameOrModel` view model is captured in the variable `message` which is passed to the layout view model.

A default template `mail/layout` is supplied is `view/layout/layout.phtml`. This template can be the starting point for your own layout template. 
The layout template can be set using the `setLayoutTemplate()` method. Alternatively,
the `mail/layout` entry in the View Manager template map can be overridden to point to your template. Another alternative is to use a factory delegator to the `MessageServiceFactory::class` to set the layout template after the Message Service is created. 

View Helpers can be used when rendering view models. A common use case is to use `$this->url()` to render a link to your application.

#### Use alternate View Resolved and View Helper Manager

LmcMail uses Service Manager aliases to get the View Resolver and View Helper Manager which resolves to the Laminas MVC resolver and manager. This allows to use any view template and helpers already defined in the application.

````php
'aliases' => [
    // These aliases are used by the MailViewRendererFactory
    // by default, they resolve to the Laminas MVC View Helper manager and Resolver
    'lmc_mail_view_helper_manager' => 'ViewHelperManager',
    'lmc_mail_view_resolver' => 'ViewResolver',
],
````
If you want to use a different resolver and view helper manager, then update the aliases to point to your classes:

````php
'aliases' => [
    'lmc_mail_view_helper_manager' => 'MyHelperManager',
    'lmc_mail_view_resolver' => 'MyViewResolver',
],
````

If you want to use your own renderer, then you can override the Service Manager factory:
````php
'factories' => [
    // Override the factory with your own
    'lmc_mail_view_renderer' => MyViewRendererFactory::class,
    /* ... */
],
````

### Event Listening

`MessageService::send()` triggers two events:  

- `MessageEvent::SEND` is triggered right before the message is sent by the transport service.
- `MessageEvent::SEND_POST` is triggered right after the message has been sent by the transport service.

The listener to these events will receive an event of class`MessageEvent` that extends the `Event` class with:

- A `$message` property containing the message. The message is also stored in an event parameter named 'message'.
- A `getMessage()` method to get the `$message` property.
- A `setMessage(Message $message)` method to set the `$message` property and the corresponding event parameters.

The `MessageService::send()` method, after triggering the `MessageEvent::SEND` event, will retrieve the message from the event and pass it to the transport service. This allows for the listener to modify the message if needed.

A typical use case for listening to the send events would be to log that a message was sent.
