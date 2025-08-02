# Laravel Telegram Notifications Channel - API Documentation

[![Latest Version on Packagist](https://img.shields.io/packagist/v/laravel-notification-channels/telegram.svg?style=flat-square)](https://packagist.org/packages/laravel-notification-channels/telegram)
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE.md)

This comprehensive API documentation covers all public APIs, functions, and components available in the Laravel Telegram Notifications Channel package.

## Table of Contents

- [Quick Start](#quick-start)
- [Core Classes](#core-classes)
  - [Telegram](#telegram-class)
  - [TelegramChannel](#telegramchannel-class)
  - [TelegramMessage](#telegrammessage-class)
  - [TelegramFile](#telegramfile-class)
  - [TelegramContact](#telegramcontact-class)
  - [TelegramLocation](#telegramlocation-class)
  - [TelegramVenue](#telegramvenue-class)
  - [TelegramPoll](#telegrampoll-class)
  - [TelegramUpdates](#telegramupdates-class)
- [Enums](#enums)
  - [ParseMode](#parsemode-enum)
  - [FileType](#filetype-enum)
- [Traits](#traits)
  - [HasSharedLogic](#hassharedlogic-trait)
- [Contracts](#contracts)
  - [TelegramSenderContract](#telegramsendercontract-interface)
- [Exceptions](#exceptions)
  - [CouldNotSendNotification](#couldnotsendnotification-exception)
- [Service Provider](#service-provider)
- [Usage Examples](#usage-examples)

## Quick Start

### Installation

```bash
composer require laravel-notification-channels/telegram
```

### Configuration

```php
// config/services.php
'telegram-bot-api' => [
    'token' => env('TELEGRAM_BOT_TOKEN', 'YOUR BOT TOKEN HERE'),
    'base_uri' => env('TELEGRAM_BOT_API_BASE_URI'), // Optional
],
```

### Basic Usage

```php
use NotificationChannels\Telegram\TelegramMessage;

public function via($notifiable)
{
    return ['telegram'];
}

public function toTelegram($notifiable)
{
    return TelegramMessage::create('Hello from Laravel!')
        ->to($notifiable->telegram_chat_id);
}
```

## Core Classes

### Telegram Class

The main class that handles direct communication with the Telegram Bot API.

#### Constructor

```php
public function __construct(
    protected ?string $token = null,
    protected HttpClient $http = new HttpClient,
    protected ?string $apiBaseUri = null
)
```

**Parameters:**
- `$token` - Telegram Bot API token
- `$http` - HTTP client instance (defaults to new GuzzleHttp\Client)
- `$apiBaseUri` - Custom API base URI (defaults to https://api.telegram.org)

#### Methods

##### `getToken(): ?string`

Returns the current bot token.

##### `setToken(string $token): self`

Sets the bot token.

**Example:**
```php
$telegram = new Telegram();
$telegram->setToken('123456789:YOUR_BOT_TOKEN');
```

##### `getApiBaseUri(): string`

Returns the current API base URI.

##### `setApiBaseUri(string $apiBaseUri): self`

Sets a custom API base URI (useful for proxies or custom endpoints).

**Example:**
```php
$telegram->setApiBaseUri('https://api.telegram.org');
```

##### `setHttpClient(HttpClient $http): self`

Sets a custom HTTP client.

##### `sendMessage(array $params): ?ResponseInterface`

Sends a text message.

**Parameters:**
- `$params` - Array of message parameters

**Example:**
```php
$response = $telegram->sendMessage([
    'chat_id' => '123456789',
    'text' => 'Hello World!',
    'parse_mode' => 'Markdown'
]);
```

**Available parameters:**
- `chat_id` (required) - Unique identifier for the target chat
- `text` (required) - Text of the message to be sent
- `parse_mode` - Send Markdown or HTML
- `disable_web_page_preview` - Disables link previews
- `disable_notification` - Sends the message silently
- `reply_to_message_id` - If the message is a reply
- `reply_markup` - Additional interface options

##### `sendFile(array $params, string $type, bool $multipart = false): ?ResponseInterface`

Sends a file (photo, document, audio, etc.).

**Parameters:**
- `$params` - Array of file parameters
- `$type` - File type (photo, document, audio, video, etc.)
- `$multipart` - Whether to use multipart form data

##### `sendPoll(array $params): ?ResponseInterface`

Sends a poll.

##### `sendContact(array $params): ?ResponseInterface`

Sends a contact.

##### `sendLocation(array $params): ?ResponseInterface`

Sends a location.

##### `sendVenue(array $params): ?ResponseInterface`

Sends a venue.

##### `getUpdates(array $params): ?ResponseInterface`

Gets bot updates.

---

### TelegramChannel Class

Laravel notification channel implementation for Telegram.

#### Constructor

```php
public function __construct(private readonly Dispatcher $dispatcher)
```

#### Methods

##### `send(mixed $notifiable, Notification $notification): ?array`

Sends the notification through the Telegram channel.

**Parameters:**
- `$notifiable` - The notifiable entity
- `$notification` - The notification instance

**Returns:** Array response from Telegram API or null

**Example:**
```php
// This method is typically called automatically by Laravel's notification system
// when you use Notification::send() or $notifiable->notify()
```

---

### TelegramMessage Class

Handles text-based Telegram notifications with support for chunking long messages.

#### Constructor

```php
public function __construct(string $content = '', public int $chunkSize = 0)
```

#### Static Methods

##### `create(string $content = ''): self`

Creates a new TelegramMessage instance.

**Example:**
```php
$message = TelegramMessage::create('Hello World!');
```

#### Instance Methods

##### `content(string $content, ?int $limit = null): self`

Sets the message content with optional character limit.

**Example:**
```php
$message->content('This is the message content', 4096);
```

##### `line(string $content): self`

Adds a line to the message content.

**Example:**
```php
$message->line('First line')
        ->line('Second line');
```

##### `lineIf(bool $condition, string $line): self`

Conditionally adds a line to the message.

**Example:**
```php
$message->lineIf($user->isActive(), 'User is active')
        ->lineIf($user->isPremium(), 'Premium user');
```

##### `escapedLine(string $content): self`

Adds a line with special characters escaped for MarkdownV2.

**Example:**
```php
$message->escapedLine('Special chars: _*[]()~`>#+-=|{}.!');
```

##### `view(string $view, array $data = [], array $mergeData = []): self`

Uses a Laravel Blade view as message content.

**Example:**
```php
$message->view('telegram.welcome', ['user' => $user]);
```

##### `chunk(int $limit = 4096): self`

Enables message chunking for long messages.

**Example:**
```php
$message->chunk(2048); // Split messages at 2048 characters
```

##### `shouldChunk(): bool`

Checks if message should be chunked.

##### `send(): array|ResponseInterface|null`

Sends the message.

**Example:**
```php
$response = $message->to('123456789')->send();
```

---

### TelegramFile Class

Handles file-based Telegram notifications with support for various file types.

#### Constructor

```php
public function __construct(string $content = '')
```

#### Static Methods

##### `create(string $content = ''): self`

Creates a new TelegramFile instance.

**Example:**
```php
$file = TelegramFile::create('File caption');
```

#### Instance Methods

##### `content(string $content): self`

Sets the file caption.

**Example:**
```php
$file->content('This is a file caption');
```

##### `file(mixed $file, FileType|string $type, ?string $filename = null): self`

Attaches a file to the message.

**Parameters:**
- `$file` - File content, path, URL, or Telegram file ID
- `$type` - File type (FileType enum or string)
- `$filename` - Optional custom filename

**Example:**
```php
// Local file
$file->file('/path/to/file.pdf', FileType::Document, 'custom-name.pdf');

// URL
$file->file('https://example.com/image.jpg', FileType::Photo);

// Telegram file ID
$file->file('BAADBAADrwADBREAAYdaWbSISxJbAg', FileType::Document);
```

##### File Type Helper Methods

```php
photo(string $file): self
audio(string $file): self
document(string $file, ?string $filename = null): self
video(string $file): self
animation(string $file): self
voice(string $file): self
videoNote(string $file): self
sticker(string $file): self
```

**Examples:**
```php
$file->photo('/path/to/image.jpg');
$file->document('/path/to/file.pdf', 'report.pdf');
$file->video('https://example.com/video.mp4');
```

##### `view(string $view, array $data = [], array $mergeData = []): self`

Uses a Laravel Blade view as file caption.

##### `hasFile(): bool`

Checks if a file is attached.

##### `send(): ?ResponseInterface`

Sends the file.

---

### TelegramContact Class

Handles contact sharing via Telegram.

#### Constructor

```php
public function __construct(string $phoneNumber = '')
```

#### Static Methods

##### `create(string $phoneNumber = ''): self`

Creates a new TelegramContact instance.

#### Instance Methods

##### `phoneNumber(string $phoneNumber): self`

Sets the contact's phone number.

**Example:**
```php
$contact->phoneNumber('+1234567890');
```

##### `firstName(string $firstName): self`

Sets the contact's first name.

##### `lastName(string $lastName): self`

Sets the contact's last name.

##### `vCard(string $vCard): self`

Sets the contact's vCard data.

**Example:**
```php
$contact = TelegramContact::create('+1234567890')
    ->firstName('John')
    ->lastName('Doe')
    ->vCard('BEGIN:VCARD\nVERSION:3.0\nFN:John Doe\nEND:VCARD');
```

##### `send(): ?ResponseInterface`

Sends the contact.

---

### TelegramLocation Class

Handles location sharing via Telegram.

#### Constructor

```php
public function __construct(float|string $latitude = '', float|string $longitude = '')
```

#### Static Methods

##### `create(float|string $latitude = '', float|string $longitude = ''): self`

Creates a new TelegramLocation instance.

#### Instance Methods

##### `latitude(float|string $latitude): self`

Sets the location's latitude.

##### `longitude(float|string $longitude): self`

Sets the location's longitude.

**Example:**
```php
$location = TelegramLocation::create(40.7128, -74.0060); // New York City
// or
$location = TelegramLocation::create()
    ->latitude(40.7128)
    ->longitude(-74.0060);
```

##### `send(): ?ResponseInterface`

Sends the location.

---

### TelegramVenue Class

Handles venue sharing via Telegram.

#### Constructor

```php
public function __construct(
    float|string $latitude = '',
    float|string $longitude = '',
    string $title = '',
    string $address = ''
)
```

#### Static Methods

##### `create(...): self`

Creates a new TelegramVenue instance with the same parameters as constructor.

#### Instance Methods

##### `latitude(float|string $latitude): self`

Sets the venue's latitude.

##### `longitude(float|string $longitude): self`

Sets the venue's longitude.

##### `title(string $title): self`

Sets the venue's name/title.

##### `address(string $address): self`

Sets the venue's address.

##### `foursquareId(string $foursquareId): self`

Sets the venue's Foursquare ID.

##### `foursquareType(string $foursquareType): self`

Sets the venue's Foursquare type.

##### `googlePlaceId(string $googlePlaceId): self`

Sets the venue's Google Places ID.

##### `googlePlaceType(string $googlePlaceType): self`

Sets the venue's Google Places type.

**Example:**
```php
$venue = TelegramVenue::create(40.7589, -73.9851, 'Times Square', 'New York, NY')
    ->foursquareId('4bd876e0f964a5203c2f3ee3')
    ->googlePlaceId('ChIJmQJIxlVYwokRLgeuocVOGVU');
```

##### `send(): ?ResponseInterface`

Sends the venue.

---

### TelegramPoll Class

Handles poll creation via Telegram.

#### Constructor

```php
public function __construct(string $question = '')
```

#### Static Methods

##### `create(string $question = ''): self`

Creates a new TelegramPoll instance.

#### Instance Methods

##### `question(string $question): self`

Sets the poll question.

##### `choices(array $choices): self`

Sets the poll answer options.

**Example:**
```php
$poll = TelegramPoll::create('What is your favorite color?')
    ->choices(['Red', 'Blue', 'Green', 'Yellow']);
```

##### `send(): ?ResponseInterface`

Sends the poll.

---

### TelegramUpdates Class

Handles fetching bot updates from Telegram.

#### Constructor

```php
public function __construct(protected array $payload = [])
```

#### Static Methods

##### `create(): self`

Creates a new TelegramUpdates instance.

#### Instance Methods

##### `limit(int $limit): self`

Sets the maximum number of updates to retrieve.

##### `options(array $options): self`

Sets additional options for the getUpdates request.

##### `latest(): self`

Gets only the latest update (sets offset to -1).

##### `get(): array`

Retrieves the updates from Telegram.

**Example:**
```php
$updates = TelegramUpdates::create()
    ->limit(10)
    ->latest()
    ->get();
```

##### `toArray(): array`

Returns the payload as an array.

---

## Enums

### ParseMode Enum

Defines available message parsing modes.

```php
enum ParseMode: string
{
    case Markdown = 'Markdown';
    case HTML = 'HTML';
    case MarkdownV2 = 'MarkdownV2';
}
```

**Usage:**
```php
$message->parseMode(ParseMode::HTML);
$message->parseMode(ParseMode::MarkdownV2);
```

### FileType Enum

Defines supported file types for Telegram.

```php
enum FileType: string
{
    case Document = 'document';
    case Photo = 'photo';
    case Audio = 'audio';
    case Video = 'video';
    case Animation = 'animation';
    case Voice = 'voice';
    case VideoNote = 'video_note';
    case Sticker = 'sticker';
}
```

#### Methods

##### `getMimeType(): string`

Returns the MIME type for the file type.

##### `getAllowedExtensions(): array`

Returns allowed file extensions for the file type.

##### `isExtensionAllowed(string $extension): bool`

Checks if a file extension is allowed for this type.

##### `toArray(): array`

Returns all file types as an array.

**Example:**
```php
$mimeType = FileType::Photo->getMimeType(); // 'image/jpeg'
$extensions = FileType::Audio->getAllowedExtensions(); // ['mp3', 'ogg', 'm4a']
$isAllowed = FileType::Video->isExtensionAllowed('mp4'); // true
```

---

## Traits

### HasSharedLogic Trait

Provides shared functionality for all Telegram message types.

#### Properties

- `$token` - Bot token override
- `$payload` - Message payload array
- `$keyboards` - Keyboard buttons array
- `$buttons` - Inline keyboard buttons array
- `$exceptionHandler` - Exception handler closure

#### Methods

##### `to(int|string $chatId): static`

Sets the recipient's chat ID.

##### `keyboardMarkup(array $markup): static`

Sets custom keyboard markup.

##### `normal(): static`

Removes parse mode (plain text).

##### `parseMode(ParseMode|string $mode): static`

Sets the message parse mode.

##### `keyboard(string $text, int $columns = 2, bool $requestContact = false, bool $requestLocation = false): static`

Adds a keyboard button.

**Example:**
```php
$message->keyboard('Contact', 2, true) // Request contact
        ->keyboard('Location', 2, false, true); // Request location
```

##### `button(string $text, string $url, int $columns = 2): static`

Adds an inline button with URL.

##### `buttonWithCallback(string $text, string $callbackData, int $columns = 2): static`

Adds an inline button with callback data.

##### `buttonWithWebApp(string $text, string $url, int $columns = 2): static`

Adds an inline button with web app.

**Example:**
```php
$message->button('Visit Website', 'https://example.com')
        ->buttonWithCallback('Click Me', 'button_clicked')
        ->buttonWithWebApp('Open App', 'https://webapp.example.com');
```

##### `disableNotification(bool $disable = true): static`

Sends the message silently.

##### `token(string $token): static`

Overrides the default bot token.

##### `hasToken(): bool`

Checks if a custom token is set.

##### `options(array $options): static`

Sets additional message options.

##### `onError(Closure $callback): self`

Registers an exception handler.

**Example:**
```php
$message->onError(function ($data) {
    Log::error('Telegram notification failed', $data);
});
```

##### `sendWhen(bool|callable $condition): static`

Sets a condition for sending the message.

##### `canSend(): bool`

Checks if the message can be sent.

##### `toNotGiven(): bool`

Checks if chat ID is not set.

##### `getPayloadValue(string $key): mixed`

Gets a value from the message payload.

##### `toArray(): array`

Returns the message payload as an array.

---

## Contracts

### TelegramSenderContract Interface

Defines the contract for classes that can send Telegram messages.

```php
interface TelegramSenderContract
{
    public function send(): ResponseInterface|array|null;
}
```

All message classes (TelegramMessage, TelegramFile, etc.) implement this interface.

---

## Exceptions

### CouldNotSendNotification Exception

Custom exception for Telegram notification failures.

#### Static Methods

##### `telegramRespondedWithAnError(ClientException $exception): self`

Creates exception for Telegram API errors.

##### `telegramBotTokenNotProvided(string $message): self`

Creates exception for missing bot token.

##### `couldNotCommunicateWithTelegram(string $message): self`

Creates exception for communication failures.

##### `fileAccessFailed(string $file): self`

Creates exception for file access failures.

##### `invalidFileIdentifier(string $file): self`

Creates exception for invalid file identifiers.

---

## Service Provider

### TelegramServiceProvider

Registers the Telegram services with Laravel's service container.

#### Methods

##### `register(): void`

Registers the Telegram class and notification channel.

**Configuration:**
The service provider automatically binds the Telegram class using configuration from:
```php
// config/services.php
'telegram-bot-api' => [
    'token' => env('TELEGRAM_BOT_TOKEN'),
    'base_uri' => env('TELEGRAM_BOT_API_BASE_URI'), // Optional
],
```

---

## Usage Examples

### Basic Text Notification

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use NotificationChannels\Telegram\TelegramMessage;

class WelcomeNotification extends Notification
{
    public function via($notifiable)
    {
        return ['telegram'];
    }

    public function toTelegram($notifiable)
    {
        return TelegramMessage::create('Welcome to our application!')
            ->to($notifiable->telegram_chat_id)
            ->parseMode(ParseMode::Markdown)
            ->line('*Thank you for joining us!*')
            ->button('Visit Website', 'https://example.com');
    }
}
```

### File Upload Notification

```php
public function toTelegram($notifiable)
{
    return TelegramFile::create('Here is your report')
        ->to($notifiable->telegram_chat_id)
        ->document('/path/to/report.pdf', 'monthly-report.pdf')
        ->button('View Online', 'https://example.com/reports');
}
```

### Location Sharing

```php
public function toTelegram($notifiable)
{
    return TelegramLocation::create(40.7128, -74.0060)
        ->to($notifiable->telegram_chat_id);
}
```

### Poll Creation

```php
public function toTelegram($notifiable)
{
    return TelegramPoll::create('What is your favorite programming language?')
        ->choices(['PHP', 'JavaScript', 'Python', 'Java'])
        ->to($notifiable->telegram_chat_id);
}
```

### Contact Sharing

```php
public function toTelegram($notifiable)
{
    return TelegramContact::create('+1234567890')
        ->firstName('John')
        ->lastName('Doe')
        ->to($notifiable->telegram_chat_id);
}
```

### Error Handling

```php
public function toTelegram($notifiable)
{
    return TelegramMessage::create('Hello!')
        ->to($notifiable->telegram_chat_id)
        ->onError(function ($data) {
            Log::error('Telegram notification failed', [
                'chat_id' => $data['to'],
                'request' => $data['request'],
                'exception' => $data['exception']->getMessage(),
            ]);
        });
}
```

### Conditional Sending

```php
public function toTelegram($notifiable)
{
    return TelegramMessage::create('Special offer!')
        ->to($notifiable->telegram_chat_id)
        ->sendWhen($notifiable->isPremium())
        ->lineIf($notifiable->hasDiscount(), 'You have a special discount!')
        ->button('Claim Offer', 'https://example.com/offer');
}
```

### Chunked Long Messages

```php
public function toTelegram($notifiable)
{
    $longContent = str_repeat('This is a very long message. ', 200);
    
    return TelegramMessage::create($longContent)
        ->to($notifiable->telegram_chat_id)
        ->chunk(4096); // Split into chunks of 4096 characters
}
```

### Custom Token

```php
public function toTelegram($notifiable)
{
    return TelegramMessage::create('Admin notification')
        ->to($notifiable->telegram_chat_id)
        ->token('DIFFERENT_BOT_TOKEN_FOR_ADMIN_BOT');
}
```

### Advanced Keyboard

```php
public function toTelegram($notifiable)
{
    return TelegramMessage::create('Choose an option:')
        ->to($notifiable->telegram_chat_id)
        ->keyboard('📞 Share Contact', 2, true)
        ->keyboard('📍 Share Location', 2, false, true)
        ->button('🌐 Website', 'https://example.com')
        ->buttonWithCallback('⚙️ Settings', 'settings_callback');
}
```

### Getting Updates

```php
use NotificationChannels\Telegram\TelegramUpdates;

// Get all updates
$updates = TelegramUpdates::create()->get();

// Get latest update only
$latestUpdate = TelegramUpdates::create()->latest()->get();

// Get limited updates
$limitedUpdates = TelegramUpdates::create()->limit(5)->get();

// Get updates with custom options
$customUpdates = TelegramUpdates::create()
    ->options(['timeout' => 30, 'allowed_updates' => ['message']])
    ->get();
```

### Direct API Usage

```php
use NotificationChannels\Telegram\Telegram;

$telegram = app(Telegram::class);

// Send a simple message
$response = $telegram->sendMessage([
    'chat_id' => '123456789',
    'text' => 'Hello from direct API!',
    'parse_mode' => 'Markdown'
]);

// Send a photo
$response = $telegram->sendFile([
    'chat_id' => '123456789',
    'photo' => 'https://example.com/image.jpg',
    'caption' => 'Check out this image!'
], 'photo');
```

This documentation covers all the public APIs, methods, and components available in the Laravel Telegram Notifications Channel package. Each section includes detailed parameter descriptions, return types, and practical examples to help developers effectively use the package.