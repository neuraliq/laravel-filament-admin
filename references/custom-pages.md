# Custom Pages & Wizards

## Settings Page

```php
namespace App\Filament\Pages;

use Filament\Forms;
use Filament\Forms\Form;
use Filament\Pages\Page;
use Filament\Actions\Action;
use Filament\Notifications\Notification;

class Settings extends Page
{
    protected static ?string $navigationIcon = 'heroicon-o-cog-6-tooth';
    protected static ?string $navigationGroup = 'System';
    protected static string $view = 'filament.pages.settings';

    public ?array $data = [];

    public function mount(): void
    {
        $this->form->fill([
            'site_name' => setting('site_name'),
            'contact_email' => setting('contact_email'),
            'maintenance_mode' => setting('maintenance_mode', false),
        ]);
    }

    public function form(Form $form): Form
    {
        return $form
            ->schema([
                Forms\Components\Section::make('General')
                    ->schema([
                        Forms\Components\TextInput::make('site_name')->required(),
                        Forms\Components\TextInput::make('contact_email')->email()->required(),
                        Forms\Components\Toggle::make('maintenance_mode'),
                    ])->columns(2),
            ])
            ->statePath('data');
    }

    public function save(): void
    {
        $data = $this->form->getState();

        foreach ($data as $key => $value) {
            setting([$key => $value]);
        }

        Notification::make()->title('Settings saved')->success()->send();
    }

    protected function getHeaderActions(): array
    {
        return [
            Action::make('save')->action('save'),
        ];
    }
}
```

## Wizard (Multi-Step Form)

```php
use Filament\Forms\Components\Wizard;

public static function form(Form $form): Form
{
    return $form->schema([
        Wizard::make([
            Wizard\Step::make('Details')
                ->schema([
                    Forms\Components\TextInput::make('name')->required(),
                    Forms\Components\TextInput::make('email')->email()->required(),
                ]),
            Wizard\Step::make('Address')
                ->schema([
                    Forms\Components\TextInput::make('street')->required(),
                    Forms\Components\TextInput::make('city')->required(),
                    Forms\Components\Select::make('country')
                        ->options(countries())
                        ->searchable()
                        ->required(),
                ]),
            Wizard\Step::make('Confirmation')
                ->schema([
                    Forms\Components\Placeholder::make('summary')
                        ->content(fn ($get) => "Creating account for {$get('name')}"),
                ]),
        ])->columnSpanFull(),
    ]);
}
```

## Custom Actions with Modals

```php
Tables\Actions\Action::make('approve')
    ->icon('heroicon-o-check')
    ->color('success')
    ->requiresConfirmation()
    ->modalHeading('Approve Order')
    ->modalDescription('This will send a confirmation email to the customer.')
    ->form([
        Forms\Components\Textarea::make('note')
            ->label('Internal note')
            ->maxLength(500),
    ])
    ->action(function (Order $record, array $data) {
        $record->approve($data['note'] ?? null);
        Notification::make()->title('Order approved')->success()->send();
    }),
```
