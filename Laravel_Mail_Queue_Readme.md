
# 📧 Laravel Mail Sending with MailHog + Job Queue

## 📥 Install & Run MailHog

1. Download the `.exe` from:  
   👉 [MailHog GitHub Releases](https://github.com/mailhog/MailHog/releases)
2. Run the `.exe` file.
3. MailHog will start and listen on:  
   👉 [http://localhost:8025](http://localhost:8025)

## ⚙️ Set Mail Config in `.env

```env
MAIL_MAILER=smtp
MAIL_HOST=127.0.0.1
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="example@test.com"
MAIL_FROM_NAME="${APP_NAME}"

QUEUE_CONNECTION=database
```

## 📌 Migrate Queue Tables

To handle queued jobs in the database:

```bash
php artisan queue:table
php artisan migrate
```

## ✉️ Create a Mailable Class

```bash
php artisan make:mail TestMail
```

**File created:** `app/Mail/TestMail.php`

**Edit the file:**

```php
<?php

namespace App\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;
use Illuminate\Queue\SerializesModels;

class TestMail extends Mailable
{
    use Queueable, SerializesModels;

    public $mailData;

    public function __construct($content)
    {
        $this->mailData = $content;
    }

    public function envelope(): Envelope
    {
        return new Envelope(subject: 'Test Mail');
    }

    public function content(): Content
    {
        return new Content(view: 'mails.Welcome');
    }

    public function attachments(): array
    {
        return [];
    }
}
```

## 📝 Create Blade Mail View

**Path:** `resources/views/mails/Welcome.blade.php`

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Test Mail</title>
</head>
<body>
    <h2>{{ $mailData['title'] }}</h2>
    <p>This is a test email sent from Laravel.</p>
    <p><strong>Message:</strong> {{ $mailData['content'] }}</p>
    <br><br>
    <p>Regards,<br>Your Laravel App</p>
</body>
</html>
```

## 📌 Create a Job Class

```bash
php artisan make:job SendEmailJob
```

**File created:** `app/Jobs/SendEmailJob.php`

**Edit the file:**

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use App\Mail\TestMail;
use Illuminate\Support\Facades\Mail;

class SendEmailJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, SerializesModels;

    protected $email;
    protected $content;

    public function __construct($email, $content)
    {
        $this->email = $email;
        $this->content = $content;
    }

    public function handle(): void
    {
        Mail::to($this->email)->send(new TestMail($this->content));
    }
}
```

## 🚀 Dispatch the Job

```php
$email = 'example@gmail.com';
$content = [
    'title' => 'Welcome via Job!',
    'content' => 'This is a test content sent via Job queue.'
];

dispatch(new SendEmailJob($email, $content));
```

## ⚙️ Start Queue Worker

```bash
php artisan queue:work
```

## ✅ Test Without Queue (Optional)

For quick, direct sending (bypassing queue):

```php
Mail::to('example@gmail.com')->send(new TestMail($content));
```

## 📌 Monitor Failed Jobs

To check failed jobs:

```bash
php artisan queue:failed
```

To retry failed jobs:

```bash
php artisan queue:retry all
```

## 🎁 Bonus: Artisan Command for Test Mail (Optional)

You can create an Artisan command to test mails easily:

```bash
php artisan make:command SendTestMailCommand
```

Then define your logic inside `app/Console/Commands/SendTestMailCommand.php`.

## 🎉 All Done!

Now you're all set to send queued emails in Laravel locally using MailHog. 🔥
