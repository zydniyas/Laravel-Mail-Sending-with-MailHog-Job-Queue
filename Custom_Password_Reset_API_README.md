
# Custom Password Reset API for Company

This API provides a custom password reset mechanism for the `companies` table in a Laravel application using RESTful APIs.

## Controller: CompanyPasswordResetController

```php
<?php

namespace App\Http\Controllers;

use App\Models\Company;
use App\Jobs\sendEmailJob;
use Illuminate\Http\Request;
use Illuminate\Support\Str;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;
use Carbon\Carbon;

class CompanyPasswordResetController extends Controller
{
    public function sendResetLinkEmail(Request $request)
    {
        $request->validate([
            'email' => 'required|email|exists:companies,Company_Email',
        ]);

        $token = Str::random(64);
        $email = $request->email;

        DB::table('password_resets')->updateOrInsert(
            ['email' => $email],
            ['email' => $email, 'token' => $token, 'created_at' => Carbon::now()]
        );

        $resetLink = url('/reset-password/' . $token);
        $content = [
            'title' => 'Password Reset Request',
            'content' => "You have requested to reset your password. Click below:

" . $resetLink,
        ];
        dispatch(new sendEmailJob($email, $content));

        return response()->json(['message' => 'Password reset link sent to your email']);
    }

    public function validateToken($token)
    {
        $resetRecord = DB::table('password_resets')
            ->where('token', $token)
            ->where('created_at', '>', Carbon::now()->subMinutes(60))
            ->first();

        if (!$resetRecord) {
            return response()->json(['message' => 'Invalid or expired token', 'valid' => false], 400);
        }

        return response()->json(['message' => 'Token is valid', 'valid' => true, 'email' => $resetRecord->email]);
    }

    public function resetPassword(Request $request)
    {
        $request->validate([
            'token' => 'required|string',
            'email' => 'required|email|exists:companies,Company_Email',
            'password' => 'required|string|min:8|confirmed',
        ]);

        $resetRecord = DB::table('password_resets')
            ->where('token', $request->token)
            ->where('email', $request->email)
            ->where('created_at', '>', Carbon::now()->subMinutes(60))
            ->first();

        if (!$resetRecord) {
            return response()->json(['message' => 'Invalid or expired token'], 400);
        }

        $company = Company::where('Company_Email', $request->email)->first();
        $company->update(['password' => Hash::make($request->password)]);

        DB::table('password_resets')->where('email', $request->email)->delete();

        $content = ['title' => 'Password Reset Successful', 'content' => 'Your password has been reset successfully.'];
        dispatch(new sendEmailJob($request->email, $content));

        return response()->json(['message' => 'Password has been reset successfully']);
    }
}
```

## API Routes

```php
Route::post('/forgot-password', [CompanyPasswordResetController::class, 'sendResetLinkEmail']);
Route::get('/reset-password/{token}', [CompanyPasswordResetController::class, 'validateToken']);
Route::post('/reset-password', [CompanyPasswordResetController::class, 'resetPassword']);
```

## Migration: create_password_resets_table

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreatePasswordResetsTable extends Migration
{
    public function up()
    {
        Schema::create('password_resets', function (Blueprint $table) {
            $table->id();
            $table->string('email')->index();
            $table->string('token');
            $table->timestamp('created_at')->nullable();
        });
    }

    public function down()
    {
        Schema::dropIfExists('password_resets');
    }
}
```

## API Usage

- **POST /api/forgot-password**
  - Body: `{ "email": "company@example.com" }`
  - Sends reset link to email.

- **GET /api/reset-password/{token}**
  - Returns token validity and associated email.

- **POST /api/reset-password**
  - Body:
  ```json
  {
    "token": "TOKEN",
    "email": "company@example.com",
    "password": "newpassword",
    "password_confirmation": "newpassword"
  }
  ```
  - Updates the password and sends a confirmation email.
