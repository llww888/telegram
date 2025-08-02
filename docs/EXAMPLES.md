# Laravel Telegram Notifications - Comprehensive Examples

This document provides comprehensive, real-world examples for using the Laravel Telegram Notifications Channel package in various scenarios.

## Table of Contents

- [Basic Examples](#basic-examples)
- [E-commerce Scenarios](#e-commerce-scenarios)
- [User Management](#user-management)
- [System Monitoring](#system-monitoring)
- [Content Management](#content-management)
- [Advanced Features](#advanced-features)
- [Integration Patterns](#integration-patterns)

## Basic Examples

### Simple Welcome Message

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use NotificationChannels\Telegram\TelegramMessage;
use NotificationChannels\Telegram\Enums\ParseMode;

class UserWelcomeNotification extends Notification
{
    public function __construct(private $user) {}

    public function via($notifiable)
    {
        return ['telegram'];
    }

    public function toTelegram($notifiable)
    {
        return TelegramMessage::create()
            ->to($notifiable->telegram_chat_id)
            ->content("🎉 Welcome to our platform, {$this->user->name}!")
            ->parseMode(ParseMode::Markdown)
            ->line('')
            ->line('Here are some things you can do:')
            ->line('• Explore our features')
            ->line('• Complete your profile')
            ->line('• Connect with others')
            ->button('Get Started', "https://yourapp.com/onboarding")
            ->button('Help Center', "https://yourapp.com/help");
    }
}

// Usage
$user->notify(new UserWelcomeNotification($user));
```

### File Upload with Progress

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use NotificationChannels\Telegram\TelegramFile;
use NotificationChannels\Telegram\Enums\FileType;

class DocumentReadyNotification extends Notification
{
    public function __construct(
        private $document,
        private $downloadUrl
    ) {}

    public function via($notifiable)
    {
        return ['telegram'];
    }

    public function toTelegram($notifiable)
    {
        return TelegramFile::create()
            ->to($notifiable->telegram_chat_id)
            ->content("📄 Your document '{$this->document->name}' is ready!")
            ->document($this->document->file_path, $this->document->original_name)
            ->button('Download from Web', $this->downloadUrl)
            ->disableNotification(false);
    }
}
```

## E-commerce Scenarios

### Order Status Updates

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use NotificationChannels\Telegram\TelegramMessage;
use NotificationChannels\Telegram\Enums\ParseMode;

class OrderStatusNotification extends Notification
{
    public function __construct(
        private $order,
        private $oldStatus,
        private $newStatus
    ) {}

    public function via($notifiable)
    {
        return ['telegram'];
    }

    public function toTelegram($notifiable)
    {
        $statusEmoji = match($this->newStatus) {
            'pending' => '⏳',
            'processing' => '🔄',
            'shipped' => '🚚',
            'delivered' => '✅',
            'cancelled' => '❌',
            default => '📦'
        };

        $message = TelegramMessage::create()
            ->to($notifiable->telegram_chat_id)
            ->parseMode(ParseMode::Markdown)
            ->content("$statusEmoji *Order #{$this->order->id} Update*")
            ->line('')
            ->line("Status changed from *{$this->oldStatus}* to *{$this->newStatus}*")
            ->line('')
            ->line("📋 *Order Details:*")
            ->line("• Total: \${$this->order->total}")
            ->line("• Items: {$this->order->items->count()}")
            ->lineIf($this->order->tracking_number, "• Tracking: `{$this->order->tracking_number}`");

        // Add status-specific information
        if ($this->newStatus === 'shipped') {
            $message->line('')
                   ->line('🚛 Your order is on its way!')
                   ->line("Expected delivery: {$this->order->estimated_delivery}");
        }

        return $message->button('View Order', "https://yourstore.com/orders/{$this->order->id}")
                      ->buttonWithCallback('Track Package', "track_{$this->order->id}");
    }
}
```

### Abandoned Cart Reminder

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use NotificationChannels\Telegram\TelegramMessage;
use NotificationChannels\Telegram\Enums\ParseMode;

class AbandonedCartNotification extends Notification
{
    public function __construct(private $cart) {}

    public function via($notifiable)
    {
        return ['telegram'];
    }

    public function toTelegram($notifiable)
    {
        $message = TelegramMessage::create()
            ->to($notifiable->telegram_chat_id)
            ->parseMode(ParseMode::Markdown)
            ->content('🛒 *You left something in your cart!*')
            ->line('')
            ->line("Don't let these great items slip away:");

        foreach ($this->cart->items as $item) {
            $message->line("• {$item->product->name} - \${$item->price}");
        }

        return $message->line('')
                      ->line("💰 *Total: \${$this->cart->total}*")
                      ->line('')
                      ->line('Complete your purchase before these items are gone!')
                      ->button('Complete Purchase', "https://yourstore.com/cart")
                      ->button('Save for Later', "https://yourstore.com/wishlist");
    }
}
```

### Inventory Alert

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use NotificationChannels\Telegram\TelegramMessage;
use NotificationChannels\Telegram\Enums\ParseMode;

class LowStockAlertNotification extends Notification
{
    public function __construct(private $product, private $currentStock) {}

    public function via($notifiable)
    {
        return ['telegram'];
    }

    public function toTelegram($notifiable)
    {
        $urgencyEmoji = $this->currentStock <= 5 ? '🚨' : '⚠️';
        
        return TelegramMessage::create()
            ->to($notifiable->telegram_chat_id)
            ->parseMode(ParseMode::Markdown)
            ->content("$urgencyEmoji *Low Stock Alert*")
            ->line('')
            ->line("*Product:* {$this->product->name}")
            ->line("*SKU:* `{$this->product->sku}`")
            ->line("*Current Stock:* {$this->currentStock} units")
            ->line("*Minimum Threshold:* {$this->product->min_stock} units")
            ->line('')
            ->lineIf($this->currentStock <= 5, '🔴 *CRITICAL: Immediate action required!*')
            ->button('Reorder Now', "https://admin.yourstore.com/products/{$this->product->id}/reorder")
            ->button('View Product', "https://admin.yourstore.com/products/{$this->product->id}");
    }
}
```

## User Management

### Account Security Alert

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use NotificationChannels\Telegram\TelegramMessage;
use NotificationChannels\Telegram\Enums\ParseMode;

class SecurityAlertNotification extends Notification
{
    public function __construct(
        private $user,
        private $action,
        private $location,
        private $timestamp
    ) {}

    public function via($notifiable)
    {
        return ['telegram'];
    }

    public function toTelegram($notifiable)
    {
        $actionEmoji = match($this->action) {
            'login' => '🔑',
            'password_change' => '🔒',
            'email_change' => '📧',
            'suspicious_activity' => '🚨',
            default => '⚠️'
        };

        return TelegramMessage::create()
            ->to($notifiable->telegram_chat_id)
            ->parseMode(ParseMode::Markdown)
            ->content("$actionEmoji *Security Alert*")
            ->line('')
            ->line("*Account:* {$this->user->email}")
            ->line("*Action:* {$this->action}")
            ->line("*Location:* {$this->location}")
            ->line("*Time:* {$this->timestamp}")
            ->line('')
            ->lineIf($this->action === 'suspicious_activity', '🚨 If this wasn\'t you, secure your account immediately!')
            ->button('Secure Account', 'https://yourapp.com/security')
            ->buttonWithCallback('Not Me', "security_alert_{$this->user->id}");
    }
}
```

### Password Reset Request

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use NotificationChannels\Telegram\TelegramMessage;
use NotificationChannels\Telegram\Enums\ParseMode;

class PasswordResetRequestNotification extends Notification
{
    public function __construct(private $resetUrl, private $expiresAt) {}

    public function via($notifiable)
    {
        return ['telegram'];
    }

    public function toTelegram($notifiable)
    {
        return TelegramMessage::create()
            ->to($notifiable->telegram_chat_id)
            ->parseMode(ParseMode::Markdown)
            ->content('🔑 *Password Reset Request*')
            ->line('')
            ->line('Someone requested a password reset for your account.')
            ->line('')
            ->line('If this was you, click the button below to reset your password:')
            ->line('')
            ->line("⏱️ This link expires at {$this->expiresAt}")
            ->line('')
            ->line('If you didn\'t request this, you can safely ignore this message.')
            ->button('Reset Password', $this->resetUrl)
            ->buttonWithCallback('Report Suspicious Activity', 'report_suspicious');
    }
}
```

## System Monitoring

### Server Health Alert

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use NotificationChannels\Telegram\TelegramMessage;
use NotificationChannels\Telegram\Enums\ParseMode;

class ServerHealthAlertNotification extends Notification
{
    public function __construct(
        private $serverName,
        private $metric,
        private $currentValue,
        private $threshold,
        private $severity
    ) {}

    public function via($notifiable)
    {
        return ['telegram'];
    }

    public function toTelegram($notifiable)
    {
        $severityEmoji = match($this->severity) {
            'critical' => '🔴',
            'warning' => '🟡',
            'info' => '🔵',
            default => '⚪'
        };

        $metricEmoji = match($this->metric) {
            'cpu' => '🖥️',
            'memory' => '💾',
            'disk' => '💿',
            'network' => '🌐',
            default => '📊'
        };

        return TelegramMessage::create()
            ->to($notifiable->telegram_chat_id)
            ->parseMode(ParseMode::Markdown)
            ->content("$severityEmoji *Server Alert - {$this->severity}*")
            ->line('')
            ->line("$metricEmoji *Server:* `{$this->serverName}`")
            ->line("*Metric:* {$this->metric}")
            ->line("*Current Value:* {$this->currentValue}")
            ->line("*Threshold:* {$this->threshold}")
            ->line("*Time:* " . now()->format('Y-m-d H:i:s'))
            ->line('')
            ->lineIf($this->severity === 'critical', '🚨 *IMMEDIATE ATTENTION REQUIRED*')
            ->button('View Dashboard', 'https://monitoring.yourapp.com')
            ->buttonWithCallback('Acknowledge', "ack_alert_{$this->serverName}_{$this->metric}");
    }
}
```

### Database Backup Status

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use NotificationChannels\Telegram\TelegramMessage;
use NotificationChannels\Telegram\Enums\ParseMode;

class DatabaseBackupNotification extends Notification
{
    public function __construct(
        private $database,
        private $status,
        private $size,
        private $duration,
        private $error = null
    ) {}

    public function via($notifiable)
    {
        return ['telegram'];
    }

    public function toTelegram($notifiable)
    {
        $statusEmoji = $this->status === 'success' ? '✅' : '❌';
        
        $message = TelegramMessage::create()
            ->to($notifiable->telegram_chat_id)
            ->parseMode(ParseMode::Markdown)
            ->content("$statusEmoji *Database Backup {$this->status}*")
            ->line('')
            ->line("*Database:* `{$this->database}`")
            ->line("*Status:* {$this->status}")
            ->line("*Duration:* {$this->duration}")
            ->lineIf($this->size, "*Size:* {$this->size}")
            ->line("*Time:* " . now()->format('Y-m-d H:i:s'));

        if ($this->error) {
            $message->line('')
                   ->line('❌ *Error Details:*')
                   ->line("```\n{$this->error}\n```");
        }

        return $message->button('View Logs', 'https://admin.yourapp.com/backups')
                      ->buttonWithCallback('Retry Backup', "retry_backup_{$this->database}");
    }
}
```

## Content Management

### Blog Post Publication

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use NotificationChannels\Telegram\TelegramMessage;
use NotificationChannels\Telegram\TelegramFile;
use NotificationChannels\Telegram\Enums\ParseMode;

class BlogPostPublishedNotification extends Notification
{
    public function __construct(private $post, private $author) {}

    public function via($notifiable)
    {
        return ['telegram'];
    }

    public function toTelegram($notifiable)
    {
        $message = TelegramMessage::create()
            ->to($notifiable->telegram_chat_id)
            ->parseMode(ParseMode::Markdown)
            ->content('📝 *New Blog Post Published!*')
            ->line('')
            ->line("*Title:* {$this->post->title}")
            ->line("*Author:* {$this->author->name}")
            ->line("*Category:* {$this->post->category->name}")
            ->line("*Published:* " . $this->post->published_at->format('M j, Y'))
            ->line('')
            ->line($this->post->excerpt)
            ->button('Read Full Article', $this->post->url)
            ->button('Share', "https://yoursite.com/share/{$this->post->slug}");

        // If there's a featured image, send it as a separate file message
        if ($this->post->featured_image) {
            return TelegramFile::create("📸 Featured image for: {$this->post->title}")
                ->to($notifiable->telegram_chat_id)
                ->photo($this->post->featured_image_url)
                ->button('Read Article', $this->post->url);
        }

        return $message;
    }
}
```

### Comment Moderation

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use NotificationChannels\Telegram\TelegramMessage;
use NotificationChannels\Telegram\Enums\ParseMode;

class CommentModerationNotification extends Notification
{
    public function __construct(private $comment, private $post) {}

    public function via($notifiable)
    {
        return ['telegram'];
    }

    public function toTelegram($notifiable)
    {
        return TelegramMessage::create()
            ->to($notifiable->telegram_chat_id)
            ->parseMode(ParseMode::Markdown)
            ->content('💬 *New Comment Awaiting Moderation*')
            ->line('')
            ->line("*Post:* {$this->post->title}")
            ->line("*Author:* {$this->comment->author_name}")
            ->line("*Email:* {$this->comment->author_email}")
            ->line("*Date:* " . $this->comment->created_at->format('M j, Y H:i'))
            ->line('')
            ->line('*Comment:*')
            ->line("```\n{$this->comment->content}\n```")
            ->buttonWithCallback('✅ Approve', "approve_comment_{$this->comment->id}")
            ->buttonWithCallback('❌ Reject', "reject_comment_{$this->comment->id}")
            ->button('View in Admin', "https://admin.yoursite.com/comments/{$this->comment->id}");
    }
}
```

## Advanced Features

### Multi-Language Support

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use NotificationChannels\Telegram\TelegramMessage;
use NotificationChannels\Telegram\Enums\ParseMode;

class MultiLanguageNotification extends Notification
{
    public function __construct(private $order) {}

    public function via($notifiable)
    {
        return ['telegram'];
    }

    public function toTelegram($notifiable)
    {
        $locale = $notifiable->preferred_language ?? 'en';
        app()->setLocale($locale);

        return TelegramMessage::create()
            ->to($notifiable->telegram_chat_id)
            ->parseMode(ParseMode::Markdown)
            ->content('*' . __('notifications.order_confirmed') . '*')
            ->line('')
            ->line('*' . __('notifications.order_number') . ':* #' . $this->order->id)
            ->line('*' . __('notifications.total_amount') . ':* $' . $this->order->total)
            ->line('*' . __('notifications.estimated_delivery') . ':* ' . $this->order->estimated_delivery)
            ->button(__('notifications.view_order'), $this->order->url)
            ->button(__('notifications.contact_support'), 'https://yourapp.com/support');
    }
}
```

### Dynamic Content with Conditions

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use NotificationChannels\Telegram\TelegramMessage;
use NotificationChannels\Telegram\Enums\ParseMode;

class DynamicUserUpdateNotification extends Notification
{
    public function __construct(private $user, private $changes) {}

    public function via($notifiable)
    {
        return ['telegram'];
    }

    public function toTelegram($notifiable)
    {
        $message = TelegramMessage::create()
            ->to($notifiable->telegram_chat_id)
            ->parseMode(ParseMode::Markdown)
            ->content('👤 *Profile Updated*')
            ->line('')
            ->line('The following changes were made to your profile:');

        foreach ($this->changes as $field => $values) {
            $fieldName = ucwords(str_replace('_', ' ', $field));
            $message->line("• *{$fieldName}:* {$values['old']} → {$values['new']}");
        }

        $message->line('')
               ->lineIf(isset($this->changes['email']), '📧 Please check your new email for verification')
               ->lineIf(isset($this->changes['phone']), '📱 SMS verification code sent to your new number')
               ->lineIf(isset($this->changes['password']), '🔐 You\'ve been logged out of other devices');

        // Add relevant buttons based on changes
        if (isset($this->changes['email'])) {
            $message->button('Verify Email', 'https://yourapp.com/verify-email');
        }

        if (isset($this->changes['password'])) {
            $message->button('Review Login Sessions', 'https://yourapp.com/security/sessions');
        }

        return $message->button('View Profile', 'https://yourapp.com/profile');
    }
}
```

### Scheduled Reminders

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use NotificationChannels\Telegram\TelegramMessage;
use NotificationChannels\Telegram\Enums\ParseMode;

class SubscriptionReminderNotification extends Notification
{
    public function __construct(
        private $subscription,
        private $daysUntilExpiry
    ) {}

    public function via($notifiable)
    {
        return ['telegram'];
    }

    public function toTelegram($notifiable)
    {
        $urgency = match(true) {
            $this->daysUntilExpiry <= 1 => ['emoji' => '🚨', 'text' => 'URGENT'],
            $this->daysUntilExpiry <= 3 => ['emoji' => '⚠️', 'text' => 'Important'],
            $this->daysUntilExpiry <= 7 => ['emoji' => '📅', 'text' => 'Reminder'],
            default => ['emoji' => 'ℹ️', 'text' => 'Notice']
        };

        $message = TelegramMessage::create()
            ->to($notifiable->telegram_chat_id)
            ->parseMode(ParseMode::Markdown)
            ->content("{$urgency['emoji']} *{$urgency['text']}: Subscription Expiring*")
            ->line('')
            ->line("*Plan:* {$this->subscription->plan_name}")
            ->line("*Expires:* " . $this->subscription->expires_at->format('M j, Y'))
            ->line("*Days Remaining:* {$this->daysUntilExpiry}");

        if ($this->daysUntilExpiry <= 3) {
            $message->line('')
                   ->line('🎯 *Don\'t lose access to:*');
            
            foreach ($this->subscription->features as $feature) {
                $message->line("• {$feature}");
            }
        }

        return $message->line('')
                      ->line('Renew now to continue enjoying uninterrupted service!')
                      ->button('Renew Subscription', "https://yourapp.com/billing/renew/{$this->subscription->id}")
                      ->button('Change Plan', 'https://yourapp.com/billing/plans')
                      ->buttonWithCallback('Remind Later', "remind_later_{$this->subscription->id}");
    }
}
```

## Integration Patterns

### Webhook Integration

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Notifications\WebhookNotification;
use NotificationChannels\Telegram\TelegramMessage;

class TelegramWebhookController extends Controller
{
    public function handle(Request $request)
    {
        $update = $request->all();
        
        // Handle callback queries (inline button presses)
        if (isset($update['callback_query'])) {
            $this->handleCallbackQuery($update['callback_query']);
        }
        
        // Handle regular messages
        if (isset($update['message'])) {
            $this->handleMessage($update['message']);
        }
        
        return response('OK');
    }
    
    private function handleCallbackQuery($callbackQuery)
    {
        $data = $callbackQuery['data'];
        $chatId = $callbackQuery['from']['id'];
        
        if (str_starts_with($data, 'approve_comment_')) {
            $commentId = str_replace('approve_comment_', '', $data);
            $this->approveComment($commentId, $chatId);
        }
        
        // Add more callback handlers...
    }
    
    private function approveComment($commentId, $chatId)
    {
        // Your comment approval logic here
        
        TelegramMessage::create('✅ Comment approved successfully!')
            ->to($chatId)
            ->send();
    }
}
```

### Queue Integration with Error Handling

```php
<?php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use NotificationChannels\Telegram\TelegramMessage;
use NotificationChannels\Telegram\Exceptions\CouldNotSendNotification;

class SendTelegramNotificationJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public $tries = 3;
    public $backoff = [10, 30, 60]; // seconds

    public function __construct(
        private $chatId,
        private $message,
        private $options = []
    ) {}

    public function handle()
    {
        try {
            $telegram = TelegramMessage::create($this->message)
                ->to($this->chatId)
                ->onError(function ($data) {
                    \Log::error('Telegram notification failed in job', $data);
                });

            // Apply any additional options
            foreach ($this->options as $method => $value) {
                if (method_exists($telegram, $method)) {
                    $telegram->$method($value);
                }
            }

            $telegram->send();
            
        } catch (CouldNotSendNotification $e) {
            \Log::error('Telegram notification job failed', [
                'chat_id' => $this->chatId,
                'message' => $this->message,
                'error' => $e->getMessage(),
                'attempt' => $this->attempts()
            ]);
            
            if ($this->attempts() >= $this->tries) {
                // Final failure - maybe notify admins?
                $this->notifyAdminsOfFailure($e);
            }
            
            throw $e; // Re-throw to trigger retry
        }
    }

    private function notifyAdminsOfFailure($exception)
    {
        // Notify administrators about the final failure
        TelegramMessage::create("🚨 Telegram notification job failed permanently")
            ->to(config('telegram.admin_chat_id'))
            ->line("Chat ID: {$this->chatId}")
            ->line("Error: {$exception->getMessage()}")
            ->send();
    }
}

// Usage
SendTelegramNotificationJob::dispatch(
    $user->telegram_chat_id,
    'Your order has been shipped!',
    ['parseMode' => 'Markdown', 'disableNotification' => false]
);
```

### Bulk Messaging with Rate Limiting

```php
<?php

namespace App\Services;

use NotificationChannels\Telegram\TelegramMessage;
use Illuminate\Support\Collection;

class TelegramBulkMessageService
{
    public function sendBulkMessage(
        Collection $recipients,
        string $message,
        array $options = [],
        int $rateLimitPerMinute = 30
    ) {
        $delay = 60 / $rateLimitPerMinute; // seconds between messages
        
        foreach ($recipients->chunk(10) as $chunk) {
            foreach ($chunk as $recipient) {
                try {
                    $telegram = TelegramMessage::create($message)
                        ->to($recipient->telegram_chat_id);
                    
                    // Apply options
                    foreach ($options as $method => $value) {
                        if (method_exists($telegram, $method)) {
                            $telegram->$method($value);
                        }
                    }
                    
                    $telegram->send();
                    
                    // Rate limiting
                    sleep($delay);
                    
                } catch (\Exception $e) {
                    \Log::error('Bulk message failed for recipient', [
                        'recipient_id' => $recipient->id,
                        'error' => $e->getMessage()
                    ]);
                }
            }
            
            // Longer pause between chunks
            sleep(5);
        }
    }
}

// Usage
$bulkService = new TelegramBulkMessageService();
$bulkService->sendBulkMessage(
    User::whereNotNull('telegram_chat_id')->get(),
    '🎉 Special announcement for all users!',
    ['parseMode' => 'Markdown'],
    20 // 20 messages per minute
);
```

This comprehensive examples guide demonstrates real-world usage patterns and advanced scenarios for the Laravel Telegram Notifications Channel package. Each example includes practical context and can be adapted to specific use cases.