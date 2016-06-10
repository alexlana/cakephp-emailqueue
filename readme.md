# CakePHP 3 - EmailQueue Plugin
This is a plugin for CakePHP 3 that let's you quickly Queue emails to be sent whenever a process function is called.

Supports **Mustache** templating and **Markdown** by default.

***WHY?***

It's not cool to bomb or hold up an order because you can't send an email confirmation. Better to queue the emails and process them in a batch later on, no?

## Install

Use Composer (sorry if you're not sure what that is yet - go learn, it's a world-changer)

```
composer require cwbit/cakephp-emailqueue:dev-master
```

### Load the plugin
Then configure your App to actually load this plugin

```php
	# in ../config/bootstrap.php
Plugin::load('EmailQueue', [
    'bootstrap' => true,        # let the plugin load its boostrap file
    'routes' => true,           # load the plugin routes file
    'ignoreMissing' => true,    # ignore missing routes or bootstrap file(s)
    'autoload' => true,      	# uncomment if you can't get composer to set the namespace/class location
    ]);
```

### Database Installation

Run the following migration command from inside your app directory to build the database for this plugin

```bash
 cd /path/to/your/app/root
 bin/cake migrations migrate --plugin EmailQueue
```

### Adjust `Configure` settings

Copy `emailqueue.sample.php` to `emailqueue.php` and change any settings you need. This is where you can set up default mail settings like `sender` and `replyTo` or even what `transport` layer you want to use as default.

If you need specific emails to send to specific people or through a specific transport layer then just make sure to set those in your `EmailTemplate` record in the database.

### Set up your Mail Templates

In your Database, set up some `EmailTemplates`, one for each type of email you want to send.

The columns in this table are modeled after the `Email::profile()` and pretty much anything that `profile` accepts can be set in the record.

You'll want one of these for each of the email types you want to send out; e.g. if you have a `password-reset` email just create a `EmailTemplate` where `email_type = password-reset`.

`message_html` and `message_text` are `TEXT` fields where you can specify the body of the email for both the `html` and `text` messages respectively.

The default `provider`(s) in `Configure::('EmailQueue.default.provider')` allow for both `Mustache` templating and `Markdown` parsing. You can make your own providers if needed and just set them in your `EmailTemplate`

# Plugin Usage
Actually sending email with EmailQueue is a simple two-step process

1. Queue the email
2. Process the email queue (CRON)

#### Queue an Email
Add the EmailQueue component to your controller

```php
	# ../src/Controller/DemoController.php

	public function initialize()
	{
		parent::initialize();

		# load the EmailQueue's EmailQueueComponent
		$this->loadComponent('EmailQueue.EmailQueue');
	}
```

And then to actually Queue an email, just specify the email **`type`**, who it's **`to`**, and any **`viewVars`** the Template (*set in the config `EmailQueue.specific.{$type}`*) will need when rendering itself.

```php
	# in your controller function
	public function someRandomFunction()
	{
		# ... do some stuff ...

        $this->EmailQueue->quickAdd('demo', 'test@user.com', ['name'=>'Test User']);

	}
```

#### Sending Queued Emails

Manually run, or add to CRON, the following commandline command

```bash
bin/cake EmailQueue.process
```

The shell has the following options:

* `--limit n` or `-l n`
  * will set the query limit() to `n` where n is an integer.
  * default `20`
* `--status foo` or `-s foo`
  * will process only emails with status `foo`
  * choice(s) : `pending|failed|sent`
  * default `pending`
* `--type foo` or `-t foo`
  * will only process emails of type `foo`
  * choice(s) are built from `Configure::read('EmailQueue.specific');`
  * default `all`
* `--id foo`
  * will only process emails with id `foo` (as long as it also matched the other filters/config settings)

All the options can be chained together.

### CLI Examples

To send all pending emails, run the following

```bash
bin/cake EmailQueue.process
```

To explicitly send all `pending` emails, run the following

```bash
bin/cake EmailQueue.process -s pending
```
To send all emails of type `order-confirmation`, run the following

```bash
bin/cake EmailQueue.process -t order-confirmation
```
To send up to `100` emails at once, run the following

```bash
bin/cake EmailQueue.process -l 100
```
To send a specific email, run the following

```bash
bin/cake EmailQueue.process --id "5869e7fd-ccf3-46c2-9b15-844335b9a86d"
```

To send up to `100`, `failed`, `user-resetpw` emails, run the following

```bash
bin/cake EmailQueue.process -l 100 -s failed -t user-resetpw
```
