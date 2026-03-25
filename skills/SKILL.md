---
name: laravel-filament-admin
description: "Filament v3 admin panel development for Laravel. Use when building admin panels, CRUD resources, custom form fields, table columns, dashboard widgets, relation managers, custom pages, or Filament actions. Triggers on tasks involving Filament resource generation, form schema, table builder, filters, bulk actions, navigation, multi-tenancy panels, or any admin dashboard with Filament. Also use when the user mentions Filament, admin panel, back-office, CMS, or wants to build an admin interface quickly. Use PROACTIVELY whenever Filament or admin panels are mentioned in a Laravel context."
compatible_agents:
  - Claude Code
  - Cursor
  - Windsurf
  - Copilot
tags:
  - laravel
  - filament
  - admin
  - crud
  - dashboard
  - forms
  - tables
  - php
---

# Filament v3 Admin Panels

Build full-featured admin panels in minutes. Filament provides form builders, table builders, dashboard widgets, and relation managers — all driven by PHP configuration arrays.

## Setup

```bash
composer require filament/filament
php artisan filament:install --panels
php artisan make:filament-user
```

## Resources (CRUD)

```bash
php artisan make:filament-resource User --generate
# Creates: UserResource.php, pages/ListUsers, CreateUser, EditUser
```

### Resource Definition

```php
namespace App\Filament\Resources;

use App\Filament\Resources\UserResource\Pages;
use App\Models\User;
use Filament\Forms;
use Filament\Forms\Form;
use Filament\Resources\Resource;
use Filament\Tables;
use Filament\Tables\Table;

class UserResource extends Resource
{
    protected static ?string $model = User::class;
    protected static ?string $navigationIcon = 'heroicon-o-users';
    protected static ?string $navigationGroup = 'User Management';
    protected static ?int $navigationSort = 1;

    public static function form(Form $form): Form
    {
        return $form->schema([
            Forms\Components\Section::make('Personal Info')->schema([
                Forms\Components\TextInput::make('name')
                    ->required()
                    ->maxLength(255),
                Forms\Components\TextInput::make('email')
                    ->email()
                    ->required()
                    ->unique(ignoreRecord: true),
                Forms\Components\DateTimePicker::make('email_verified_at'),
            ])->columns(2),

            Forms\Components\Section::make('Security')->schema([
                Forms\Components\TextInput::make('password')
                    ->password()
                    ->revealable()
                    ->dehydrateStateUsing(fn ($state) => bcrypt($state))
                    ->dehydrated(fn (?string $state) => filled($state))
                    ->required(fn (string $operation) => $operation === 'create'),
                Forms\Components\Select::make('role')
                    ->options([
                        'admin' => 'Admin',
                        'editor' => 'Editor',
                        'user' => 'User',
                    ])
                    ->required(),
                Forms\Components\Toggle::make('is_active')
                    ->default(true),
            ])->columns(2),
        ]);
    }

    public static function table(Table $table): Table
    {
        return $table
            ->columns([
                Tables\Columns\TextColumn::make('name')
                    ->searchable()
                    ->sortable(),
                Tables\Columns\TextColumn::make('email')
                    ->searchable()
                    ->copyable(),
                Tables\Columns\TextColumn::make('role')
                    ->badge()
                    ->color(fn (string $state) => match ($state) {
                        'admin' => 'danger',
                        'editor' => 'warning',
                        'user' => 'success',
                    }),
                Tables\Columns\IconColumn::make('is_active')
                    ->boolean(),
                Tables\Columns\TextColumn::make('created_at')
                    ->dateTime()
                    ->sortable()
                    ->toggleable(isToggledHiddenByDefault: true),
            ])
            ->filters([
                Tables\Filters\SelectFilter::make('role')
                    ->options([
                        'admin' => 'Admin',
                        'editor' => 'Editor',
                        'user' => 'User',
                    ]),
                Tables\Filters\TernaryFilter::make('is_active'),
            ])
            ->actions([
                Tables\Actions\EditAction::make(),
                Tables\Actions\DeleteAction::make(),
            ])
            ->bulkActions([
                Tables\Actions\BulkActionGroup::make([
                    Tables\Actions\DeleteBulkAction::make(),
                ]),
            ])
            ->defaultSort('created_at', 'desc');
    }

    public static function getRelations(): array
    {
        return [
            // RelationManagers\PostsRelationManager::class,
        ];
    }

    public static function getPages(): array
    {
        return [
            'index' => Pages\ListUsers::route('/'),
            'create' => Pages\CreateUser::route('/create'),
            'edit' => Pages\EditUser::route('/{record}/edit'),
        ];
    }
}
```

## Dashboard Widgets

```php
namespace App\Filament\Widgets;

use App\Models\Order;
use Filament\Widgets\StatsOverviewWidget;
use Filament\Widgets\StatsOverviewWidget\Stat;

class StatsOverview extends StatsOverviewWidget
{
    protected function getStats(): array
    {
        return [
            Stat::make('Total Revenue', '$' . number_format(Order::sum('total'), 2))
                ->description('32% increase')
                ->descriptionIcon('heroicon-m-arrow-trending-up')
                ->color('success')
                ->chart([7, 3, 4, 5, 6, 3, 5, 8]),
            Stat::make('New Customers', \App\Models\User::whereMonth('created_at', now()->month)->count())
                ->description('This month')
                ->color('info'),
            Stat::make('Pending Orders', Order::where('status', 'pending')->count())
                ->description('Needs attention')
                ->color('warning'),
        ];
    }
}
```

## Relation Managers

```php
namespace App\Filament\Resources\UserResource\RelationManagers;

use Filament\Forms;
use Filament\Forms\Form;
use Filament\Resources\RelationManagers\RelationManager;
use Filament\Tables;
use Filament\Tables\Table;

class PostsRelationManager extends RelationManager
{
    protected static string $relationship = 'posts';

    public function form(Form $form): Form
    {
        return $form->schema([
            Forms\Components\TextInput::make('title')->required(),
            Forms\Components\RichEditor::make('body')->required()->columnSpanFull(),
            Forms\Components\Toggle::make('is_published'),
        ]);
    }

    public function table(Table $table): Table
    {
        return $table
            ->columns([
                Tables\Columns\TextColumn::make('title')->searchable(),
                Tables\Columns\IconColumn::make('is_published')->boolean(),
                Tables\Columns\TextColumn::make('created_at')->dateTime(),
            ])
            ->headerActions([Tables\Actions\CreateAction::make()])
            ->actions([Tables\Actions\EditAction::make(), Tables\Actions\DeleteAction::make()]);
    }
}
```

## Key Rules

1. Always use `->searchable()` and `->sortable()` on columns users will filter by
2. Always use `->unique(ignoreRecord: true)` on edit forms — prevents self-conflict on unique fields
3. Always use `->dehydrated(fn (?string $state) => filled($state))` on password fields — prevents blanking on edit
4. Always use `->columns(2)` on sections — single-column forms waste horizontal space
5. Use `->toggleable(isToggledHiddenByDefault: true)` for less-important columns
6. Always use `->badge()->color()` for status columns — visual clarity
7. Use `->copyable()` on emails, IDs, and tokens — saves users clicks
8. Always provide `->defaultSort()` on tables — unsorted tables feel broken
9. Use Relation Managers for hasMany/belongsToMany — keep related CRUD in context
10. Use `--generate` flag when creating resources — auto-generates form/table from migration
