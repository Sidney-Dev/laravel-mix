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

### signature
The signature property defines the command name and any input expectations such as arguments and options. 

`protected $signature = "greet {user} {language}";`

Arguments can also be made optional with a question mark right after it
`protected $signature = "mail:send {user?}"`

Arguments can also be optional with default value
`protected $signature = "mail:send {user=foo}"`

#### signature - arguments

Each signature argument can be accessed using the argument method of the command class.

`$this->argument('user');`
`$this->argument('language');`

// the following retreives all the arguments in an array
`$this->arguments()`

#### signature - option
There are options with values and options without value.

// option without value
`protected $signature = "mail:send {user?} {--queue}"`
If the queue is passed as an option, the queue argument is set to true.

// the equal sign indicates that the user may pass a value to the option
`protected $signature = "mail:send {user?} {--queue=}"`

// creating a shortcut for the option
`'mail:send {user} {--Q|queue}'`
`php artisan mail:send 1 -Qdefault`

### input descriptions
`protected $signature = 'emails:send {user : The ID of each user} {--queue : Whether the job should be queued}';`

options are retrieved using the option method of the class
`$this->option('queue)`

## exit code
We can manually specified the exit code when the command runs
// outputs an error message
`$this->error('Something went wrong.');`

// outputs an error message and stops the execution with exit code 1
`$this->fail('Something went wrong.');`


# Closure Commands - routes/console.php

the console.php defines console based entry points into a laravel application.

## Usage

the command method accepts two arguments: the command signature, and a closure which receives the command arguments and options including any dependencies.

// the closure accepts any type-hint dependencies
`Artisan::command('mail:send {user}', function($user) {`

`})->purpose('Send some email from here');`

the command class has a "purpose" method that can be chained, which will be the command description.


# Isolatable commands

To ensure that only one instance of the command run implement the Illuminate\Contracts\Console\Isolatable interface on the command class

`class SendEmails extends Commands implements Isolatable`

laravel uses a cache locking mechanism to create a lock. However, the locking duration can be customized.

The isolotable can also be called when writing the command via the prompt:
`php artisan mail:send 1 --isolated` // this will return 0 exit code even if the command does not run

to specify the exit code to be returned if it is not able to execute write the following:

`php artisan mail:send 1 --isolated=12`

## Lock expiration
Isolated locks expire after the command is finished. Or if the command is interrupted and unable to finish, the lock will expire after one hour. However, the lock expiration time may be adjusted.

### a) via artisan console

`php artisan report:generate --lock-expiration=300`

### b) via command class
public function isolationLockExpiresAt(): DateTimeInterface|DateInterval
{
    return now()->addMinutes(5);
}

# Input Arrays

# Prompting for Missing input
To prevent the command to fail if a required argument is not present, we can implement
`Illuminate\Contracts\Console\PromptsForMissingInput` in the command class

This will prompt the user to enter the required input/s.

And to customise the prompt message for each argument, we can implement the promptForMissingArgumentsUsing method

`protected function promptForMissingArgumentsUsing(): array`
`{`
`    return [`
`        'user' => ['Which user ID should receive the mail?', 'placeholder text'],` optional placeholder
`    ];`
`}`

# Prompting for input