# Writing commands

create a new artisan command using `php artisan make:command SendEmails`

Commands are typically stored in app/Console/Commands/SendEmails.php

For better reusability, it is best practice to keep any logic separate from the command. The artisan class should only be used for execution.

## Command class structure
The command class ships with the following properties and methods

1. signature: a protected property that allows the definition of the command's input expectations
2. description: describes the function of the command
3. handle(): the method called when the command is executed. Accepts any dependencies

Ideally, this class should not contain any logic, but be a trigger to other tasks/services.

### signature property - $this->argument(argument name)
The signature property defines the command input expectations and can accept required and optional arguments.

`protected $signature = "greet {user} {language}";`

Each signature argument can be accessed using the argument method of the command class.

$this->argument('user');
$this->argument('language');

## exit code
We can manually specified the exit code when the command runs
$this->error('Something went wrong.'); // outputs an error message
$this->fail('Something went wrong.'); // outputs an error message and stops the execution with exit code 1


## Closure Commands - routes/console.php

the console.php defines console based entry points into a laravel application.

## definition
the command method accepts two arguments: the command signature, and a closure which receives the command arguments and options including any dependencies.

Artisan::command('mail:send {user}', function($user) {

})->purpose('Send some email from here');

the command class has a "purpose" method that can be chained, which will be the command description.

# Isolatable commands

Sometimes you may wish to ensure that only onse instance of a command can run at a time. Implement the Illuminate\Contracts\Console\Isolatable interface on the command class

laravel uses a cache locking mechanism to create a lock. However, the locking duration can be customized.

php artisan report:generate --lock-expiration=300


# Defining Input Expectations

// optional argument
protected $signature = "mail:sned {user?}"

// optional argument with default value
protected $signature = "mail:sned {user=foo}"

# Command options
There are options with values and options without value.

// option without value
protected $signature = "mail:sned {user?} {--queue}"
If the queue is passed as an option, the queue argument is set to true.

// the equal sign indicates that the user may pass a value to the option
protected $signature = "mail:sned {user?} {--queue=}"

// creatin g a shortcut for the option
'mail:send {user} {--Q|queue}'
php artisan mail:send 1 -Qdefault