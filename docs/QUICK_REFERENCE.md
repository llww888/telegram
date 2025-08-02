# Laravel Telegram Notifications - Quick Reference

A concise reference guide for the Laravel Telegram Notifications Channel package.

## Installation & Setup

```bash
composer require laravel-notification-channels/telegram
```

```php
// config/services.php
'telegram-bot-api' => [
    'token' => env('TELEGRAM_BOT_TOKEN'),
    'base_uri' => env('TELEGRAM_BOT_API_BASE_URI'), // Optional
],
```

## Quick Start

```php
use NotificationChannels\Telegram\TelegramMessage;

public function via($notifiable) { return ['telegram']; }

public function toTelegram($notifiable)
{
    return TelegramMessage::create('Hello!')
        ->to($notifiable->telegram_chat_id);
}
```

## Core Classes Summary

| Class | Purpose | Key Methods |
|-------|---------|-------------|
| `Telegram` | Direct API communication | `sendMessage()`, `sendFile()`, `getUpdates()` |
| `TelegramChannel` | Laravel notification channel | `send()` |
| `TelegramMessage` | Text messages with chunking | `create()`, `content()`, `line()`, `chunk()` |
| `TelegramFile` | File attachments | `create()`, `file()`, `photo()`, `document()` |
| `TelegramContact` | Contact sharing | `create()`, `phoneNumber()`, `firstName()` |
| `TelegramLocation` | Location sharing | `create()`, `latitude()`, `longitude()` |
| `TelegramVenue` | Venue sharing | `create()`, `title()`, `address()` |
| `TelegramPoll` | Poll creation | `create()`, `question()`, `choices()` |
| `TelegramUpdates` | Fetch bot updates | `create()`, `limit()`, `latest()`, `get()` |

## Message Building (Shared Methods)

### Basic Setup
```php
->to(chatId)                          // Set recipient
->token('custom_token')               // Override bot token
->parseMode(ParseMode::Markdown)      // Set parse mode
->normal()                            // Remove parse mode
->disableNotification(true)           // Silent message
->options(['key' => 'value'])         // Additional options
```

### Content Building
```php
->content('message text')             // Set main content
->line('add line')                    // Add line
->lineIf(condition, 'text')           // Conditional line
->escapedLine('text with *special*')  // Escaped line
->view('blade.view', $data)           // Use Blade view
```

### Keyboards & Buttons
```php
// Regular keyboards
->keyboard('Button Text', columns: 2, requestContact: false, requestLocation: false)

// Inline buttons
->button('Text', 'https://url.com')                    // URL button
->buttonWithCallback('Text', 'callback_data')          // Callback button
->buttonWithWebApp('Text', 'https://webapp.com')       // Web app button

// Custom keyboard
->keyboardMarkup(['keyboard' => [[['text' => 'Button']]]])
```

### Conditions & Error Handling
```php
->sendWhen(condition)                 // Conditional sending
->canSend()                           // Check if can send
->onError(function($data) { ... })    // Error handler
```

## File Types & Methods

### TelegramFile Methods
```php
TelegramFile::create('Caption')
    ->file($file, FileType::Document, 'filename.ext')  // Generic file
    ->photo('path/url/file_id')                         // Photo
    ->audio('path/url/file_id')                         // Audio
    ->document('path', 'filename.pdf')                  // Document
    ->video('path/url/file_id')                         // Video
    ->animation('path/url/file_id')                     // GIF/Animation
    ->voice('path/url/file_id')                         // Voice note
    ->videoNote('path/url/file_id')                     // Video note
    ->sticker('path/url/file_id')                       // Sticker
```

### File Sources
```php
// Local file
->document('/path/to/file.pdf')

// URL
->photo('https://example.com/image.jpg')

// Telegram file ID
->document('BAADBAADrwADBREAAYdaWbSISxJbAg')

// Resource or Stream
->document($fileResource, FileType::Document, 'name.pdf')
```

## Enums

### ParseMode
```php
ParseMode::Markdown     // Basic markdown
ParseMode::HTML         // HTML formatting
ParseMode::MarkdownV2   // Advanced markdown
```

### FileType
```php
FileType::Document      // Documents
FileType::Photo         // Images
FileType::Audio         // Audio files
FileType::Video         // Video files
FileType::Animation     // GIFs
FileType::Voice         // Voice messages
FileType::VideoNote     // Video notes
FileType::Sticker       // Stickers
```

## Contact & Location

### TelegramContact
```php
TelegramContact::create('+1234567890')
    ->firstName('John')
    ->lastName('Doe')
    ->vCard('BEGIN:VCARD...')
```

### TelegramLocation
```php
TelegramLocation::create(latitude, longitude)
    ->latitude(40.7128)
    ->longitude(-74.0060)
```

### TelegramVenue
```php
TelegramVenue::create(lat, lng, 'Title', 'Address')
    ->foursquareId('id')
    ->googlePlaceId('id')
```

## Polls

```php
TelegramPoll::create('Question?')
    ->choices(['Option 1', 'Option 2', 'Option 3'])
```

## Updates & Bot Data

```php
TelegramUpdates::create()
    ->limit(10)                       // Limit results
    ->latest()                        // Get only latest
    ->options(['timeout' => 30])      // Custom options
    ->get()                           // Fetch updates
```

## Direct API Usage

```php
$telegram = app(Telegram::class);

// Send message
$telegram->sendMessage([
    'chat_id' => '123',
    'text' => 'Hello',
    'parse_mode' => 'Markdown'
]);

// Send file
$telegram->sendFile([
    'chat_id' => '123',
    'photo' => 'https://example.com/image.jpg'
], 'photo');

// Get updates
$telegram->getUpdates(['limit' => 10]);
```

## Advanced Features

### Message Chunking
```php
TelegramMessage::create($longText)
    ->chunk(4096)                     // Enable chunking at 4096 chars
    ->shouldChunk()                   // Check if chunking enabled
```

### Conditional Logic
```php
$message = TelegramMessage::create('Base message')
    ->lineIf($user->isVip(), '⭐ VIP User')
    ->sendWhen($user->allowsNotifications())
    ->when($order->isUrgent(), function($msg) {
        return $msg->line('🚨 URGENT ORDER');
    });
```

### Error Handling
```php
->onError(function($data) {
    Log::error('Telegram failed', [
        'chat_id' => $data['to'],
        'request' => $data['request'],
        'exception' => $data['exception']
    ]);
})
```

## Utility Methods

### Payload & Data
```php
->getPayloadValue('key')              // Get payload value
->toArray()                           // Get payload as array
->toNotGiven()                        // Check if no recipient
->hasToken()                          // Check custom token
->hasFile()                           // Check if file attached (TelegramFile)
```

### FileType Utilities
```php
FileType::Photo->getMimeType()                    // Get MIME type
FileType::Audio->getAllowedExtensions()           // Get allowed extensions
FileType::Video->isExtensionAllowed('mp4')        // Check extension
FileType::toArray()                               // All types as array
```

## Common Patterns

### Notification Class Template
```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use NotificationChannels\Telegram\TelegramMessage;
use NotificationChannels\Telegram\Enums\ParseMode;

class MyNotification extends Notification
{
    public function __construct(private $data) {}

    public function via($notifiable) { return ['telegram']; }

    public function toTelegram($notifiable)
    {
        return TelegramMessage::create()
            ->to($notifiable->telegram_chat_id)
            ->parseMode(ParseMode::Markdown)
            ->content('*Your message here*')
            ->line('Additional info')
            ->button('Action', 'https://example.com');
    }
}
```

### On-Demand Notification
```php
use Illuminate\Support\Facades\Notification;

Notification::route('telegram', $chatId)
    ->notify(new MyNotification($data));
```

### Multiple Recipients
```php
$users = User::whereNotNull('telegram_chat_id')->get();
Notification::send($users, new MyNotification($data));
```

## Exception Handling

### Exception Types
```php
CouldNotSendNotification::telegramRespondedWithAnError($exception)
CouldNotSendNotification::telegramBotTokenNotProvided($message)
CouldNotSendNotification::couldNotCommunicateWithTelegram($message)
CouldNotSendNotification::fileAccessFailed($file)
CouldNotSendNotification::invalidFileIdentifier($file)
```

### Try-Catch Pattern
```php
try {
    $message->send();
} catch (CouldNotSendNotification $e) {
    Log::error('Telegram notification failed: ' . $e->getMessage());
}
```

## Configuration Examples

### Environment Variables
```bash
TELEGRAM_BOT_TOKEN=123456789:ABCdefGHIjklMNOpqrsTUVwxyz
TELEGRAM_BOT_API_BASE_URI=https://api.telegram.org  # Optional
```

### Service Configuration
```php
// config/services.php
'telegram-bot-api' => [
    'token' => env('TELEGRAM_BOT_TOKEN'),
    'base_uri' => env('TELEGRAM_BOT_API_BASE_URI'),
],
```

### Custom Configuration
```php
// Custom Telegram instance
$telegram = new Telegram('custom_token', new HttpClient(), 'https://custom.api.url');
```

This quick reference provides all essential methods and patterns for effective use of the Laravel Telegram Notifications Channel package.