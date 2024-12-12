Dependencies:

XAMPP : https://www.apachefriends.org/download.html
Composer: https://getcomposer.org/Composer-Setup.exe

Start XAMPP

Go to C:\xampp\htdocs
Create a new Folder and name it 
Open vscode and open the same project directory C:\xampp\htdocs\ProjectName
Open terminal and type:
composer create-project laravel/laravel ProjectName


Now create the views:

register.blade.php
----------------------------------------------------------------------------------------
<!DOCTYPE html>
<html>
<head>
    <title>Sign-Up</title>
</head>
<body>
    <h1>Register</h1>
    <form method="POST" action="/register">
        @csrf
        <label for="name">Name:</label>
        <input type="text" id="name" name="name" required>
        <br>
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required>
        <br>
        <label for="password">Password:</label>
        <input type="password" id="password" name="password" required>
        <br>
        <button type="submit">Register</button>
    </form>
</body>
</html>
----------------------------------------------------------------------------------------

login.blade.php
----------------------------------------------------------------------------------------
<!DOCTYPE html>
<html>
<head>
    <title>Login</title>
</head>
<body>
    <h1>Login</h1>
    @if ($errors->any())
        <p style="color:red">{{ $errors->first('error') }}</p>
    @endif
    <form method="POST" action="/login">
        @csrf
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required>
        <br>
        <label for="password">Password:</label>
        <input type="password" id="password" name="password" required>
        <br>
        <button type="submit">Login</button>
    </form>
</body>
</html>
----------------------------------------------------------------------------------------

dashboard.blade.php
----------------------------------------------------------------------------------------

<!DOCTYPE html>
<html>
<head>
    <title>Dashboard</title>
</head>
<body>
    <h1>Welcome, {{ $user['name'] }}</h1>
    <p>Email: {{ $user['email'] }}</p>
    <a href="/logout">Logout</a>
</body>
</html>


----------------------------------------------------------------------------------------


Making a controller
Type in terminal

php artisan make:controller AuthController


app\Http\Controllers\AuthController.php
----------------------------------------------------------------------------------------
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class AuthController extends Controller
{
// Show the registration form
public function showRegisterForm()
{
    return view('register');
}

// Handle the registration
public function register(Request $request)
{
    // Temporary in-memory list of users
    $users = session('users', []);

    // Validate the form input
    $validatedData = $request->validate([
        'name' => 'required|string|max:255',
        'email' => 'required|email|unique:users,email',
        'password' => 'required|string|min:6',
    ]);

    // Add new user to the users list
    $users[] = [
        'name' => $validatedData['name'],
        'email' => $validatedData['email'],
        'password' => bcrypt($validatedData['password']),  // Hash the password
    ];

    // Store the list of users in the session
    session(['users' => $users]);

    // Redirect to login page after successful registration
    return redirect('/login');
}

// Show the login form
public function showLoginForm()
{
    return view('login');
}

// Handle the login
public function login(Request $request)
{
    // Retrieve users from the session
    $users = session('users', []);

    // Validate the input
    $validatedData = $request->validate([
        'email' => 'required|email',
        'password' => 'required|string',
    ]);

    // Check if credentials match any user
    foreach ($users as $user) {
        if ($user['email'] === $validatedData['email'] && $user['password'] === $validatedData['password']) {
            // Store the logged-in user in the session
            session(['logged_in_user' => $user]);

            // Redirect to the dashboard
            return redirect('/dashboard');
        }
    }

    // If no match, return back with an error message
    return back()->withErrors(['error' => 'Invalid credentials']);
}

// Show the dashboard
public function showDashboard()
{
    // Retrieve the logged-in user from the session
    $user = session('logged_in_user');

    // If no user is logged in, redirect to login page
    if (!$user) {
        return redirect('/login');
    }

    // Return the dashboard view with the user's data
    return view('dashboard', ['user' => $user]);
}

// Logout the user
public function logout()
{
    // Clear the logged_in_user session
    session()->forget('logged_in_user');

    // Redirect to login page
    return redirect('/login');
}
}
----------------------------------------------------------------------------------------

Define Routes

routes\web.php

----------------------------------------------------------------------------------------
<?php

use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return view('welcome');
});

Route::get('/register', [AuthController::class, 'showRegisterForm']);
Route::post('/register', [AuthController::class, 'register']);

Route::get('/login', [AuthController::class, 'showLoginForm']);
Route::post('/login', [AuthController::class, 'login']);

Route::get('/dashboard', [AuthController::class, 'showDashboard']);
Route::get('/logout', [AuthController::class, 'logout']);
----------------------------------------------------------------------------------------

Keep in mind that there are some pieces of code here that we have not discussed in the lecture like Form validation logic and Bcrypt logic in public function register, do research on your own for this as they will come in quiz



