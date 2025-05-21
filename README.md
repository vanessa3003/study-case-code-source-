<?php
/* ===================================================
COMPLETE TEACHER MANAGEMENT SYSTEM - SINGLE FILE SOLUTION
(Laravel 12 + Filament 3)
=================================================== */

// 1. MODEL (app/Models/Teacher.php)
namespace App\Models;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Facades\Crypt;

class Teacher extends Model {
    protected $fillable = ['name', 'nip', 'email', 'phone', 'subject', 'join_date'];
    protected $casts = ['join_date' => 'date'];

    // Auto-encrypt/decrypt NIP
    public function setNipAttribute($value) {
        $this->attributes['nip'] = Crypt::encryptString($value);
    }
    public function getNipAttribute($value) {
        try { return Crypt::decryptString($value); } 
        catch (\Exception $e) { return $value; }
    }
}

// 2. MIGRATION (database/migrations/YYYY_MM_DD_create_teachers_table.php)
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateTeachersTable extends Migration {
    public function up() {
        Schema::create('teachers', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->text('nip'); // Stores encrypted data
            $table->string('email')->unique();
            $table->string('phone')->nullable();
            $table->string('subject');
            $table->date('join_date');
            $table->timestamps();
            $table->softDeletes();
        });
    }
    public function down() {
        Schema::dropIfExists('teachers');
    }
}

// 3. FILAMENT RESOURCE (app/Filament/Resources/TeacherResource.php)
namespace App\Filament\Resources;
use App\Models\Teacher;
use Filament\Forms;
use Filament\Tables;
use Filament\Resources\Resource;

class TeacherResource extends Resource {
    protected static string $model = Teacher::class;
    protected static string $navigationIcon = 'heroicon-o-academic-cap';
    protected static string $navigationGroup = 'School Data';

    public static function form(Forms\Form $form): Forms\Form {
        return $form->schema([
            Forms\Components\Section::make('Teacher Information')
                ->schema([
                    Forms\Components\TextInput::make('name')->required(),
                    Forms\Components\TextInput::make('nip')->label('Teacher ID')->required(),
                    Forms\Components\TextInput::make('email')->email()->required(),
                    Forms\Components\TextInput::make('phone')->tel(),
                    Forms\Components\TextInput::make('subject')->required(),
                    Forms\Components\DatePicker::make('join_date')->required(),
                ])->columns(2)
        ]);
    }

    public static function table(Tables\Table $table): Tables\Table {
        return $table
            ->columns([
                Tables\Columns\TextColumn::make('name')->searchable()->sortable(),
                Tables\Columns\TextColumn::make('nip')->label('ID'),
                Tables\Columns\TextColumn::make('email')->searchable(),
                Tables\Columns\TextColumn::make('subject'),
                Tables\Columns\TextColumn::make('join_date')->date(),
            ])
            ->filters([Tables\Filters\TrashedFilter::make()])
            ->actions([Tables\Actions\EditAction::make()])
            ->bulkActions([Tables\Actions\DeleteBulkAction::make()]);
    }
}
