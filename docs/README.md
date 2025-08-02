# Laravel Telegram Notifications Channel - Documentation

Comprehensive documentation for the Laravel Telegram Notifications Channel package, covering all public APIs, functions, and components.

## 📚 Documentation Overview

This documentation suite provides complete coverage of the Laravel Telegram Notifications Channel package, designed to help developers effectively integrate Telegram notifications into their Laravel applications.

### 🎯 What's Included

- **Complete API Reference** - Detailed documentation of all classes, methods, and components
- **Real-World Examples** - Practical scenarios and implementation patterns
- **Quick Reference Guide** - Concise method reference for daily development
- **Usage Instructions** - Step-by-step guides for common use cases

## 📖 Documentation Files

### [📋 API Documentation](./API_DOCUMENTATION.md)
**Complete reference for all public APIs, functions, and components**

- Core classes and their methods
- Enums, traits, and interfaces
- Service provider configuration
- Exception handling
- Detailed parameter descriptions
- Return types and examples

**Best for:** Understanding the complete API surface and detailed method documentation.

### [💡 Comprehensive Examples](./EXAMPLES.md)
**Real-world examples and practical scenarios**

- Basic notification examples
- E-commerce scenarios (orders, cart abandonment, inventory)
- User management (security alerts, password resets)
- System monitoring (server health, backups)
- Content management (blog posts, comments)
- Advanced features (multi-language, dynamic content)
- Integration patterns (webhooks, queues, bulk messaging)

**Best for:** Learning through practical examples and implementing specific use cases.

### [⚡ Quick Reference](./QUICK_REFERENCE.md)
**Concise reference guide for daily development**

- Installation and setup
- Core classes summary
- Method quick reference
- Common patterns and templates
- Configuration examples
- Utility methods

**Best for:** Quick lookups during development and as a cheat sheet.

## 🚀 Getting Started

### 1. Installation
```bash
composer require laravel-notification-channels/telegram
```

### 2. Configuration
```php
// config/services.php
'telegram-bot-api' => [
    'token' => env('TELEGRAM_BOT_TOKEN'),
],
```

### 3. Basic Usage
```php
use NotificationChannels\Telegram\TelegramMessage;

public function toTelegram($notifiable)
{
    return TelegramMessage::create('Hello from Laravel!')
        ->to($notifiable->telegram_chat_id);
}
```

## 🏗️ Package Architecture

The package is organized around several core concepts:

### Core Components
- **`Telegram`** - Main API client for direct Telegram Bot API communication
- **`TelegramChannel`** - Laravel notification channel implementation
- **Message Types** - Specialized classes for different content types
- **Shared Logic** - Common functionality through traits and base classes

### Message Types
- **`TelegramMessage`** - Text messages with formatting and chunking
- **`TelegramFile`** - File attachments (photos, documents, audio, video)
- **`TelegramContact`** - Contact information sharing
- **`TelegramLocation`** - Geographic location sharing
- **`TelegramVenue`** - Venue/place information
- **`TelegramPoll`** - Interactive polls
- **`TelegramUpdates`** - Bot update fetching

### Supporting Components
- **Enums** - Type-safe constants (ParseMode, FileType)
- **Contracts** - Interface definitions
- **Exceptions** - Structured error handling
- **Traits** - Shared functionality (HasSharedLogic)

## 🔧 Key Features

### ✨ Rich Message Types
Support for all major Telegram message types including text, files, locations, contacts, polls, and venues.

### 🎛️ Flexible API
Fluent interface for building complex messages with keyboards, buttons, and formatting.

### 📱 Laravel Integration
Seamless integration with Laravel's notification system and service container.

### 🔄 Message Chunking
Automatic splitting of long messages to comply with Telegram's message limits.

### ⚡ Error Handling
Comprehensive exception handling with custom callbacks and retry mechanisms.

### 🔧 Extensible
Support for custom tokens, API endpoints, and HTTP clients.

## 📋 Method Categories

### Message Building
- Content management (`content()`, `line()`, `view()`)
- Formatting (`parseMode()`, `escapedLine()`)
- Conditional logic (`lineIf()`, `sendWhen()`)

### User Interface
- Keyboards (`keyboard()`, `keyboardMarkup()`)
- Inline buttons (`button()`, `buttonWithCallback()`)
- Web apps (`buttonWithWebApp()`)

### File Handling
- Multiple file types (`photo()`, `document()`, `video()`)
- Flexible sources (local files, URLs, Telegram file IDs)
- Metadata support (filenames, captions)

### Configuration
- Recipients (`to()`, routing)
- Bot settings (`token()`, API endpoints)
- Message options (`disableNotification()`, `options()`)

### Advanced Features
- Error handling (`onError()`)
- Chunking (`chunk()`, `shouldChunk()`)
- Conditional sending (`canSend()`, `sendWhen()`)

## 🎨 Usage Patterns

### Laravel Notifications
```php
class OrderShipped extends Notification
{
    public function via($notifiable) { return ['telegram']; }
    
    public function toTelegram($notifiable)
    {
        return TelegramMessage::create("Order #{$this->order->id} shipped!")
            ->to($notifiable->telegram_chat_id)
            ->button('Track Package', $this->order->tracking_url);
    }
}
```

### Direct API Usage
```php
$telegram = app(Telegram::class);
$telegram->sendMessage([
    'chat_id' => '123456789',
    'text' => 'Hello World!',
    'parse_mode' => 'Markdown'
]);
```

### File Sharing
```php
TelegramFile::create('Your invoice is ready')
    ->to($user->telegram_chat_id)
    ->document($invoicePath, 'invoice.pdf')
    ->send();
```

## 🔍 Finding What You Need

### For API Details
➡️ [API Documentation](./API_DOCUMENTATION.md) - Complete method signatures, parameters, and return types

### For Implementation Ideas
➡️ [Examples Guide](./EXAMPLES.md) - Real-world scenarios and advanced patterns

### For Quick Lookups
➡️ [Quick Reference](./QUICK_REFERENCE.md) - Concise method reference and common patterns

## 🤝 Contributing

This documentation is maintained alongside the Laravel Telegram Notifications Channel package. For contributions, issues, or questions:

- **Package Repository:** [laravel-notification-channels/telegram](https://github.com/laravel-notification-channels/telegram)
- **Documentation Issues:** Report documentation issues in the main repository

## 📄 License

This documentation is provided under the same MIT license as the Laravel Telegram Notifications Channel package.

---

**Happy coding!** 🚀 This documentation covers everything you need to effectively use Telegram notifications in your Laravel applications.