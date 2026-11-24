# UMB Library Email Service

PHP endpoint for sending Primo call number and location information by email, including delivery to an email-to-SMS gateway address. The script uses PHPMailer and Amazon SES SMTP.

## Requirements

- A PHP-enabled web server with outbound SMTP access.
- Composer and the `phpmailer/phpmailer` package.
- Amazon SES SMTP credentials and a sender configured for the service.

## Setup

The script loads `../vendor/autoload.php`. Install PHPMailer in the parent directory of this repository so the deployed layout is:

```text
parent-directory/
├── vendor/
│   └── autoload.php
└── email/
    ├── README.md
    └── email.php
```

From the parent directory, run:

```sh
composer require phpmailer/phpmailer
```

Configure these environment variables for the PHP server process:

| Variable | Value |
| --- | --- |
| `SMTP_USERNAME` | Amazon SES SMTP username |
| `SMTP_PASSWORD` | Amazon SES SMTP password |

The script reads these values with `getenv()`; it does not load a `.env` file. Keep credentials out of source control.

The following settings are currently defined in `email.php`:

| Setting | Current value |
| --- | --- |
| SMTP host | `email-smtp.us-east-1.amazonaws.com` |
| SMTP port | `587` |
| SMTP security | `tls` |
| From and reply-to address | `Library.technology@umb.edu` |
| Sender display name | `Library-DoNotReply` |

Update these settings if your deployment uses another SMTP service or sender.

## Request format

Send a POST request with a JSON body to the deployed `email.php` endpoint.

| Field | Type | Purpose |
| --- | --- | --- |
| `to` | string | Recipient email or email-to-SMS gateway address |
| `message` | string | Call number or primary message content |
| `details` | string | Location or additional information |
| `gCaptchaResponse` | string | CAPTCHA response token, when CAPTCHA is enabled |

Example (replace the URL and recipient before running; this sends two messages):

```sh
curl --request POST 'https://your-server.example/email/email.php' \
  --header 'Content-Type: application/json' \
  --data '{
    "to": "recipient@example.com",
    "message": "Call number: QA76.73.P98",
    "details": "Location: Library stacks"
  }'
```

The endpoint sends `message` first and `details` second as separate emails. It does not currently set a subject. Successful requests return no response body; caught mail errors and failed CAPTCHA verification return HTTP 500.

## Optional CAPTCHA

CAPTCHA is disabled by default (`$enableCaptcha = false`). To enable it, set `$enableCaptcha` to `true`, configure `$gCaptchaSecret` securely on the server, and have the client include `gCaptchaResponse` in every request. Do not commit a real CAPTCHA secret.

## Deployment notes

The current script permits any CORS origin, has CAPTCHA disabled, and does not implement caller authentication or rate limiting. Restrict access and configure appropriate abuse controls before exposing it publicly.

Validate PHP syntax after changes:

```sh
php -l email.php
```
