<?php
namespace App\Models;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Facades\Crypt;

class Teacher extends Model {
    protected $fillable = ['name', 'nip', 'email', 'phone', 'subject', 'join_date'];
    protected $casts = ['join_date' => 'date'];

    // Encrypt NIP automatically
    public function setNipAttribute($value) {
        $this->attributes['nip'] = Crypt::encryptString($value);
    }
    public function getNipAttribute($value) {
        try { return Crypt::decryptString($value); } 
        catch (\Exception $e) { return $value; }
    }
}

