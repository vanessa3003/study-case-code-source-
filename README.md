# study-case-code-source-
vanessa Dangremont_uts_20230801025 study case code source 
// ===============================================
// 1. INSTALLATION STEPS
// ===============================================
/*
composer create-project laravel/laravel teacher-management
cd teacher-management

# Install Filament
composer require filament/filament:"^3.0"
php artisan filament:install
php artisan migrate

# Install Laravel Breeze for authentication (optional)
composer require laravel/breeze --dev
php artisan breeze:install
npm install && npm run dev
php artisan migrate
*/

// ===============================================
// 2. MODEL: app/Models/Teacher.php
// ===============================================

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Teacher extends Model
{
    use HasFactory;

    protected $fillable = [
        'name', 'nip', 'email', 'phone',
    ];
}

// ===============================================
// 3. MIGRATION: database/migrations/xxxx_xx_xx_create_teachers_table.php
// ===============================================

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void {
        Schema::create('teachers', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('nip')->unique();
            $table->string('email')->unique();
            $table->string('phone')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void {
        Schema::dropIfExists('teachers');
    }
};

// ===============================================
// 4. FILAMENT RESOURCE: app/Filament/Resources/TeacherResource.php
// ===============================================

namespace App\Filament\Resources;

use App\Filament\Resources\TeacherResource\Pages;
use App\Models\Teacher;
use Filament\Forms;
use Filament\Tables;
use Filament\Resources\Resource;

class TeacherResource extends Resource
{
    protected static ?string $model = Teacher::class;
    protected static ?string $navigationIcon = 'heroicon-o-academic-cap';
    protected static ?string $navigationGroup = 'Master Data';

    public static function form(Forms\Form $form): Forms\Form
    {
        return $form
            ->schema([
                Forms\Components\TextInput::make('name')
                    ->required()->maxLength(255),

                Forms\Components\TextInput::make('nip')
                    ->required()->unique(ignoreRecord: true),

                Forms\Components\TextInput::make('email')
                    ->required()->email()->unique(ignoreRecord: true),

                Forms\Components\TextInput::make('phone')
                    ->tel()->maxLength(15),
            ]);
    }

    public static function table(Tables\Table $table): Tables\Table
    {
        return $table
            ->columns([
                Tables\Columns\TextColumn::make('name')->searchable()->sortable(),
                Tables\Columns\TextColumn::make('nip'),
                Tables\Columns\TextColumn::make('email'),
                Tables\Columns\TextColumn::make('phone'),
            ])
            ->filters([])
            ->actions([
                Tables\Actions\EditAction::make(),
                Tables\Actions\DeleteAction::make(),
            ])
            ->bulkActions([
                Tables\Actions\DeleteBulkAction::make(),
            ]);
    }

    public static function getPages(): array
    {
        return [
            'index' => Pages\ManageTeachers::route('/'),
        ];
    }
}

// ===============================================
// 5. FILAMENT PAGE: app/Filament/Resources/TeacherResource/Pages/ManageTeachers.php
// ===============================================

namespace App\Filament\Resources\TeacherResource\Pages;

use App\Filament\Resources\TeacherResource;
use Filament\Resources\Pages\ManageRecords;

class ManageTeachers extends ManageRecords
{
    protected static string $resource = TeacherResource::class;
}

// ===============================================
// 6. OPTIONAL SECURITY (Extra) — Encryption Example
// ===============================================

use Illuminate\Support\Facades\Crypt;

// Encrypting NIP before saving
$encryptedNip = Crypt::encryptString($request->nip);

// Decrypting later
$originalNip = Crypt::decryptString($teacher->nip);

// ===============================================
// 7. AUTHENTICATION (Laravel Breeze or Jetstream handles login)
// ===============================================
// User can only access Filament panel after login:
// Filament automatically adds middleware for auth in panel route group.

