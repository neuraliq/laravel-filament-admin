# Actions, Bulk Operations & Notifications

## Table Actions

```php
->actions([
    Tables\Actions\ActionGroup::make([
        Tables\Actions\ViewAction::make(),
        Tables\Actions\EditAction::make(),
        Tables\Actions\Action::make('duplicate')
            ->icon('heroicon-o-document-duplicate')
            ->action(fn (Product $record) => $record->replicate()->save())
            ->successNotificationTitle('Product duplicated'),
        Tables\Actions\DeleteAction::make()
            ->requiresConfirmation(),
    ]),
])
```

## Bulk Actions

```php
->bulkActions([
    Tables\Actions\BulkActionGroup::make([
        Tables\Actions\DeleteBulkAction::make(),
        Tables\Actions\BulkAction::make('export')
            ->icon('heroicon-o-arrow-down-tray')
            ->action(function (Collection $records) {
                return response()->streamDownload(function () use ($records) {
                    echo $records->toCsv();
                }, 'export.csv');
            }),
        Tables\Actions\BulkAction::make('updateStatus')
            ->icon('heroicon-o-arrow-path')
            ->form([
                Forms\Components\Select::make('status')
                    ->options(['active' => 'Active', 'inactive' => 'Inactive'])
                    ->required(),
            ])
            ->action(function (Collection $records, array $data) {
                $records->each->update(['status' => $data['status']]);
            })
            ->deselectRecordsAfterCompletion(),
    ]),
])
```

## Notifications

```php
use Filament\Notifications\Notification;

// Simple notification
Notification::make()
    ->title('Order saved')
    ->success()
    ->send();

// With body and action
Notification::make()
    ->title('New order received')
    ->body("Order #{$order->number} — \${$order->total}")
    ->icon('heroicon-o-shopping-cart')
    ->actions([
        \Filament\Notifications\Actions\Action::make('view')
            ->url(OrderResource::getUrl('edit', ['record' => $order]))
            ->button(),
        \Filament\Notifications\Actions\Action::make('dismiss')
            ->close(),
    ])
    ->sendToDatabase($admins);

// Duration
Notification::make()
    ->title('Processing...')
    ->info()
    ->duration(5000) // 5 seconds
    ->send();
```

## Header Actions (Page Level)

```php
// On List page
protected function getHeaderActions(): array
{
    return [
        Actions\CreateAction::make(),
        Actions\Action::make('import')
            ->icon('heroicon-o-arrow-up-tray')
            ->form([
                Forms\Components\FileUpload::make('file')
                    ->acceptedFileTypes(['text/csv'])
                    ->required(),
            ])
            ->action(fn (array $data) => ImportJob::dispatch($data['file'])),
    ];
}
```
