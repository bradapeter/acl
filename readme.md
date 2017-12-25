

### What We'll Build

When building an application, we often need to set up an access control list (ACL). An ACL specifies the level of permission granted to a user of an application. For example a user John may have the permission to read and write to a resource while another user Smith may have the permission only to read the resource.

In this tutorial, I will teach you how to add access control to a Laravel app using [Laravel-permission package](https://github.com/spatie/laravel-permission). For this tutorial we will build a simple blog application where users can be assigned different levels of permission. Our user admin page will look like this:
<div class="notification m-b-md">Related Course: [Getting Started with JavaScript for Web Development](https://bit.ly/2rVqDcs)</div>

![](https://cdn.scotch.io/21141/oPohpGscSXi0p2Fni011_admin.png)

### Why Use Laravel-Permission

The Laravel-Permission package is built on top of Laravel's authorization features introduced in the 5.1.1 release. Although there are other packages that claim to offer similar functionalities, none of them have the same level of activity and maintenance as the laravel-permission package.

### Development Environment and Installation

You can get Laravel up and running by first downloading the installer

    composer global require <span class="token string">"laravel/installer"</span>`</pre>

    Then add `$HOME/.composer/vendor/bin` to your $PATH so the `laravel` executable can be located by your system. Now you can install the latest stable version of Laravel by running

    <pre data-title="bash" class=" language-bash">`laravel new`</pre>

    ![](https://cdn.scotch.io/21141/YLzkKppZT3ayJA5lCYkB_install.png)

    To install the laravel-permission package run

    <pre data-title="bash" class=" language-bash">`composer require spatie/laravel-permission`</pre>

    Next include the package to our list of service providers, in `config/app.php` add `Spatie\Permission\PermissionServiceProvider::class` so our file looks like this

    <pre data-title="php" class=" language-php">`<span class="token string">'providers'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">[</span>
        <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
        Spatie\<span class="token package">Permission<span class="token punctuation">\</span>PermissionServiceProvider</span><span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token keyword">class</span><span class="token punctuation">,</span>
    <span class="token punctuation">]</span><span class="token punctuation">;</span>`</pre>

    Next publish the migration file for this package with the command

    <pre data-title="bash" class=" language-bash">`php artisan vendor:publish --provider<span class="token operator">=</span><span class="token string">"Spatie\Permission\PermissionServiceProvider"</span> --tag<span class="token operator">=</span><span class="token string">"migrations"</span>`</pre>

    ### Database Setup and Migrations

    Next create the database and update the `.env` file to include the database information. For example, for this tutorial the database information section of the `.env` looks like this:

    <pre data-title="php" class=" language-php">`<span class="token constant">DB_CONNECTION</span><span class="token operator">=</span>mysql
    <span class="token constant">DB_HOST</span><span class="token operator">=</span><span class="token number">127.0</span><span class="token punctuation">.</span><span class="token number">0.1</span>
    <span class="token constant">DB_PORT</span><span class="token operator">=</span><span class="token number">3306</span>
    <span class="token constant">DB_DATABASE</span><span class="token operator">=</span>acl4
    <span class="token constant">DB_USERNAME</span><span class="token operator">=</span>root
    <span class="token constant">DB_PASSWORD</span><span class="token operator">=</span>`</pre>

    To build the tables, run

    <pre data-title="bash" class=" language-bash">`php artisan migrate`</pre>

    Please note that in Laravel 5.4 the default character set is changed to utf8mb4, therefore if you are running MariaDB or MYSQL version lower than 5.7.7 you may get this error when trying to run migration files

    <pre data-title="sql" class=" language-sql">`[Illuminate\Database\QueryException]
    SQLSTATE[42000]: Syntax error or access violation: 1071 Specified key was too long; max key length is 767 bytes (SQL: alter table users add unique users_email_unique(email))

    [PDOException]
    SQLSTATE[42000]: Syntax error or access violation: 1071 Specified key was too long; max key length is 767 bytes`</pre>

    To fix this error edit the `app\Providers\AppServiceProvider.php` file, setting the default string length in the boot method

    <pre data-title="php" class=" language-php">`<span class="token keyword">use</span> <span class="token package">Illuminate<span class="token punctuation">\</span>Support<span class="token punctuation">\</span>Facades<span class="token punctuation">\</span>Schema</span><span class="token punctuation">;</span>

    <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">boot</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
    <span class="token punctuation">{</span>
        Schema<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">defaultStringLength</span><span class="token punctuation">(</span><span class="token number">191</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>`</pre>

    After that run the migration again. If it works as normal you would find the following tables in your database:

*   `migrations`: This keeps track of migration process that have ran
*   `users`: This holds the users data of the application
*   `password_resets`: Holds token information when users request a new password
*   `permissions`: This holds the various permissions needed in the application
*   `roles`: This holds the roles in our application
*   `role_has_permission`: This is a pivot table that holds relationship information between the permissions table and the role table
*   `user_has_roles`: Also a pivot table, holds relationship information between the roles and the users table.
*   `user_has_permissions`: Also a pivot table, holds relationship information between the users table and the permissions table.

    Publish the configuration file for this package by running

    <pre data-title="bash" class=" language-bash">`php artisan vendor:publish --provider<span class="token operator">=</span><span class="token string">"Spatie\Permission\PermissionServiceProvider"</span> --tag<span class="token operator">=</span><span class="token string">"config"</span>`</pre>

    The config file allows us to set the location of the Eloquent model of the permission and role class. You can also manually set the table names that should be used to retrieve your roles and permissions. Next we need to add the `HasRoles` trait to the User model:

    <pre data-title="php" class=" language-php">`<span class="token keyword">use</span> <span class="token package">Illuminate<span class="token punctuation">\</span>Foundation<span class="token punctuation">\</span>Auth<span class="token punctuation">\</span>User</span> <span class="token keyword">as</span> Authenticatable<span class="token punctuation">;</span>
    <span class="token keyword">use</span> <span class="token package">Spatie<span class="token punctuation">\</span>Permission<span class="token punctuation">\</span>Traits<span class="token punctuation">\</span>HasRoles</span><span class="token punctuation">;</span>

    <span class="token keyword">class</span> <span class="token class-name">User</span> <span class="token keyword">extends</span> <span class="token class-name">Authenticatable</span> <span class="token punctuation">{</span>
        <span class="token keyword">use</span> <span class="token package">HasRoles</span><span class="token punctuation">;</span>

        <span class="token comment">// ...</span>
    <span class="token punctuation">}</span>`</pre>

    ### Laravel Collective HTML Form builder

    Next install Laravel Collective HTML Form builder as this will be useful further on when we are creating our forms:

    <pre data-title="bash" class=" language-bash">`composer require laravelcollective/html`</pre>

    Then add your new provider to the providers array of `config/app.php`:

    <pre data-title="php" class=" language-php">`<span class="token string">'providers'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">[</span>
        <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
        Collective\<span class="token package">Html<span class="token punctuation">\</span>HtmlServiceProvider</span><span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token keyword">class</span><span class="token punctuation">,</span>
    <span class="token punctuation">]</span><span class="token punctuation">;</span>`</pre>

    Finally, add two class aliases to the aliases array of `config/app.php`:

    <pre data-title="php" class=" language-php">`<span class="token string">'aliases'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">[</span>
         <span class="token comment">// ...</span>
         <span class="token string">'Form'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> Collective\<span class="token package">Html<span class="token punctuation">\</span>FormFacade</span><span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token keyword">class</span><span class="token punctuation">,</span>
         <span class="token string">'Html'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> Collective\<span class="token package">Html<span class="token punctuation">\</span>HtmlFacade</span><span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token keyword">class</span><span class="token punctuation">,</span>
         <span class="token comment">// ...</span>
         <span class="token punctuation">]</span><span class="token punctuation">,</span> `</pre>

    That's all the installation and configuration needed. A role can be created like a regular Eloquent model, like this:

    <pre data-title="php" class=" language-php">`<span class="token keyword">use</span> <span class="token package">Spatie<span class="token punctuation">\</span>Permission<span class="token punctuation">\</span>Models<span class="token punctuation">\</span>Role</span><span class="token punctuation">;</span>
    <span class="token keyword">use</span> <span class="token package">Spatie<span class="token punctuation">\</span>Permission<span class="token punctuation">\</span>Models<span class="token punctuation">\</span>Permission</span><span class="token punctuation">;</span>

    <span class="token variable">$role</span> <span class="token operator">=</span> Role<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token string">'name'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'writer'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token variable">$permission</span> <span class="token operator">=</span> Permission<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token string">'name'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'edit articles'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>`</pre>

    You can also get the permissions associated to a user like this:

    <pre data-title="php" class=" language-php">`<span class="token variable">$permissions</span> <span class="token operator">=</span> <span class="token variable">$user</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">permissions</span><span class="token punctuation">;</span>`</pre>

    And using the [pluck method, `pluck()`](https://laravel.com/docs/5.4/collections#method-pluck) you can get the role names associated with a user like this:

    <pre data-title="php" class=" language-php">`<span class="token variable">$roles</span> <span class="token operator">=</span> <span class="token variable">$user</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">roles</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">pluck</span><span class="token punctuation">(</span><span class="token string">'name'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>`</pre>

    Other methods available to us include:

*   `givePermissionTo()`: Allows us to give persmission to a user or role
*   `revokePermissionTo()`: Revoke permission from a user or role
*   `hasPermissionTo()`: Check if a user or role has a given permission
*   `assignRole()`: Assigns role to a user
*   `removeRole()`: Removes role from a user
*   `hasRole()`: Checks if a user has a role
*   `hasAnyRole(Role::all())`: Checks if a user has any of a given list of roles
*   `hasAllRoles(Role::all())`: Checks if a user has all of a given list of role

    The methods `assignRole`, `hasRole`, `hasAnyRole`, `hasAllRoles` and `removeRole` can accept a string, a `Spatie\Permission\Models\Role-object` or an `\Illuminate\Support\Collection` object. The `givePermissionTo` and `revokePermissionTo` methods can accept a string or a `Spatie\Permission\Models\Permission` object. 

    Laravel-Permission also allows to use Blade directives to verify if the logged in user has all or any of a given list of roles:

    <pre data-title="blade" class=" language-blade">`@role('writer')
        I'm a writer!
    @else
        I'm not a writer...
    @endrole

    @hasrole('writer')
        I'm a writer!
    @else
        I'm not a writer...
    @endhasrole

    @hasanyrole(Role::all())
        I have one or more of these roles!
    @else
        I have none of these roles...
    @endhasanyrole

    @hasallroles(Role::all())
        I have all of these roles!
    @else
        I don't have all of these roles...
    @endhasallroles`</pre>

    The Blade directives above depends on the users role. Sometimes we need to check directly in our view if a user has a certain permission. You can do that using Laravel's native `@can` directive:

    <pre data-title="blade" class=" language-blade">`@can('Edit Post')
        I have permission to edit
    @endcan`</pre>

    ### Controllers, Authentication and Views

    You will need a total of four controllers for this application. Let's use resource controllers, as this automatically adds stub methods for us. Our controllers will be called

1.  PostController
2.  UserController
3.  RoleController
4.  PermissionController

    Before working on these controllers let's create our authentication system. With one command Laravel provides a quick way to scaffold all of the routes and views needed for authentication.

    <pre data-title="bash" class=" language-bash">`php artisan make:auth`</pre>

    After running this command you would notice two new links for user login and registration in the home page. 

    ![](https://cdn.scotch.io/21141/lv0V3dQ8TnSR87DgERxs_auth.png)

    This command also creates a `HomeController` (you can delete this as it won't be needed), a `resources/views/layouts/app.blade.php` file which contains markup that would be shared by all our views and an `app/Http/Controllers/Auth` directory which contains the controllers for registration and login. Switch into this directory and open the `RegisterController.php`file. Remove the `bcrypt` function in the `create` method, so the the method looks like this

    <pre data-title="php" class=" language-php">`<span class="token keyword">protected</span> <span class="token keyword">function</span> <span class="token function">create</span><span class="token punctuation">(</span><span class="token keyword">array</span> <span class="token variable">$data</span><span class="token punctuation">)</span>
    <span class="token punctuation">{</span>
        <span class="token keyword">return</span> User<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">[</span>
            <span class="token string">'name'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token variable">$data</span><span class="token punctuation">[</span><span class="token string">'name'</span><span class="token punctuation">]</span><span class="token punctuation">,</span>
            <span class="token string">'email'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token variable">$data</span><span class="token punctuation">[</span><span class="token string">'email'</span><span class="token punctuation">]</span><span class="token punctuation">,</span>
            <span class="token string">'password'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token variable">$data</span><span class="token punctuation">[</span><span class="token string">'password'</span><span class="token punctuation">]</span><span class="token punctuation">,</span>
        <span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>  `</pre>

    Instead let's define a mutator in `app\User.php` which would encrypt all our password fields. In `app\User.php` add this method:

    <pre data-title="php" class=" language-php">`<span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">setPasswordAttribute</span><span class="token punctuation">(</span><span class="token variable">$password</span><span class="token punctuation">)</span>
    <span class="token punctuation">{</span>   
        <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">attributes</span><span class="token punctuation">[</span><span class="token string">'password'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token function">bcrypt</span><span class="token punctuation">(</span><span class="token variable">$password</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>`</pre>

    This would provide the same functionality as before but now you don't need to write the `bcrypt` function when dealing with the password field in subsequent controllers.

    Also in the `RegisterController.php`file. Change the `$redirectTo` property to:

    <pre data-title="php" class=" language-php">`<span class="token keyword">protected</span> <span class="token variable">$redirectTo</span> <span class="token operator">=</span> <span class="token string">'/'</span><span class="token punctuation">;</span>`</pre>

    Do the same thing in the `LoginController.php`file. 

    Since the `HomeController` has been deleted our users are now redirected to the home page which would contain a list of our blog posts.

    Next let's edit the `resources/views/layouts/app.blade.php` file to include: an extra drop-down 'Admin' link to view all users and an errors file which checks if our form produced any error. The 'Admin' link would only be viewed by users with the 'Admin' Role. We would also create a custom `styles.css` which would have extra styling for our `resources/views/posts/index.blade.php` view. The styling is just a paragraph in the teaser of our index view, the file should be located in `public/css/styles.css` 

    <pre data-title="html" class=" language-html">`{{-- resources/views/layouts/app.blade.php --}}
    <span class="token doctype">&lt;!DOCTYPE html&gt;</span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>html</span> <span class="token attr-name">lang</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ config(<span class="token punctuation">'</span>app.locale<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>head</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>meta</span> <span class="token attr-name">charset</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>utf-8<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>meta</span> <span class="token attr-name">http-equiv</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>X-UA-Compatible<span class="token punctuation">"</span></span> <span class="token attr-name">content</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>IE=edge<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>meta</span> <span class="token attr-name">name</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>viewport<span class="token punctuation">"</span></span> <span class="token attr-name">content</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>width=device-width, initial-scale=1<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>

        <span class="token comment">&lt;!-- CSRF Token --&gt;</span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>meta</span> <span class="token attr-name">name</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>csrf-token<span class="token punctuation">"</span></span> <span class="token attr-name">content</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ csrf_token() }}<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>title</span><span class="token punctuation">&gt;</span></span>{{ config('app.name', 'Laravel') }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>title</span><span class="token punctuation">&gt;</span></span>

        <span class="token comment">&lt;!-- Styles --&gt;</span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>link</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ asset(<span class="token punctuation">'</span>css/app.css<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span> <span class="token attr-name">rel</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>stylesheet<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>link</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ asset(<span class="token punctuation">'</span>css/styles.css<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span> <span class="token attr-name">rel</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>stylesheet<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>link</span> <span class="token attr-name">rel</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>stylesheet<span class="token punctuation">"</span></span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css<span class="token punctuation">"</span></span> <span class="token attr-name">integrity</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u<span class="token punctuation">"</span></span> <span class="token attr-name">crossorigin</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>anonymous<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>

        <span class="token comment">&lt;!-- Scripts --&gt;</span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>script</span><span class="token punctuation">&gt;</span></span>
            window.Laravel = {!! json_encode([
                'csrfToken' =&gt; csrf_token(),
            ]) !!};
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>script</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>script</span> <span class="token attr-name">src</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>https://use.fontawesome.com/9712be8772.js<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>script</span><span class="token punctuation">&gt;</span></span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>head</span><span class="token punctuation">&gt;</span></span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>body</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">id</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>app<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>nav</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>navbar navbar-default navbar-static-top<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>container<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>navbar-header<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>

                        <span class="token comment">&lt;!-- Collapsed Hamburger --&gt;</span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>button</span> <span class="token attr-name">type</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>button<span class="token punctuation">"</span></span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>navbar-toggle collapsed<span class="token punctuation">"</span></span> <span class="token attr-name">data-toggle</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>collapse<span class="token punctuation">"</span></span> <span class="token attr-name">data-target</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>#app-navbar-collapse<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>span</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>sr-only<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>Toggle Navigation<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>span</span><span class="token punctuation">&gt;</span></span>
                            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>span</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>icon-bar<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>span</span><span class="token punctuation">&gt;</span></span>
                            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>span</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>icon-bar<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>span</span><span class="token punctuation">&gt;</span></span>
                            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>span</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>icon-bar<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>span</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>button</span><span class="token punctuation">&gt;</span></span>

                        <span class="token comment">&lt;!-- Branding Image --&gt;</span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>navbar-brand<span class="token punctuation">"</span></span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ url(<span class="token punctuation">'</span>/<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                            {{ config('app.name', 'Laravel') }}
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>collapse navbar-collapse<span class="token punctuation">"</span></span> <span class="token attr-name">id</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>app-navbar-collapse<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                        <span class="token comment">&lt;!-- Left Side Of Navbar --&gt;</span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>ul</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>nav navbar-nav<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>li</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ url(<span class="token punctuation">'</span>/<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>Home<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>li</span><span class="token punctuation">&gt;</span></span>
                            @if (!Auth::guest())
                                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>li</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ route(<span class="token punctuation">'</span>posts.create<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>New Article<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>li</span><span class="token punctuation">&gt;</span></span>
                             @endif
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>ul</span><span class="token punctuation">&gt;</span></span>

                        <span class="token comment">&lt;!-- Right Side Of Navbar --&gt;</span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>ul</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>nav navbar-nav navbar-right<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                            <span class="token comment">&lt;!-- Authentication Links --&gt;</span>
                            @if (Auth::guest())
                                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>li</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ route(<span class="token punctuation">'</span>login<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>Login<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>li</span><span class="token punctuation">&gt;</span></span>
                                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>li</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ route(<span class="token punctuation">'</span>register<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>Register<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>li</span><span class="token punctuation">&gt;</span></span>
                            @else
                                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>li</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>dropdown<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>#<span class="token punctuation">"</span></span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>dropdown-toggle<span class="token punctuation">"</span></span> <span class="token attr-name">data-toggle</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>dropdown<span class="token punctuation">"</span></span> <span class="token attr-name">role</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>button<span class="token punctuation">"</span></span> <span class="token attr-name">aria-expanded</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>false<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                                        {{ Auth::user()-&gt;name }} <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>span</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>caret<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>span</span><span class="token punctuation">&gt;</span></span>
                                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span>

                                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>ul</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>dropdown-menu<span class="token punctuation">"</span></span> <span class="token attr-name">role</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>menu<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>li</span><span class="token punctuation">&gt;</span></span>
                                            @role('Admin') {{-- Laravel-permission blade helper --}}
                                            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>#<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>i</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>fa fa-btn fa-unlock<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>i</span><span class="token punctuation">&gt;</span></span>Admin<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span>
                                            @endrole
                                            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ route(<span class="token punctuation">'</span>logout<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span>
                                                <span class="token attr-name">onclick</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>event.preventDefault();
                                                         document.getElementById(<span class="token punctuation">'</span>logout-form<span class="token punctuation">'</span>).submit();<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                                                Logout
                                            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span>

                                            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>form</span> <span class="token attr-name">id</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>logout-form<span class="token punctuation">"</span></span> <span class="token attr-name">action</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ route(<span class="token punctuation">'</span>logout<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span> <span class="token attr-name">method</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>POST<span class="token punctuation">"</span></span> <span class="token attr-name">style</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>display: none;<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                                                {{ csrf_field() }}
                                            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>form</span><span class="token punctuation">&gt;</span></span>
                                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>li</span><span class="token punctuation">&gt;</span></span>
                                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>ul</span><span class="token punctuation">&gt;</span></span>
                                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>li</span><span class="token punctuation">&gt;</span></span>
                            @endif
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>ul</span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>nav</span><span class="token punctuation">&gt;</span></span>

            @if(Session::has('flash_message'))
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>container<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>      
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>alert alert-success<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>em</span><span class="token punctuation">&gt;</span></span> {!! session('flash_message') !!}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>em</span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
            @endif 

            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>row<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>col-md-8 col-md-offset-2<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>              
                    @include ('errors.list') {{-- Including error file --}}
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

            @yield('content')

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

        <span class="token comment">&lt;!-- Scripts --&gt;</span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>script</span> <span class="token attr-name">src</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ asset(<span class="token punctuation">'</span>js/app.js<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>script</span><span class="token punctuation">&gt;</span></span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>body</span><span class="token punctuation">&gt;</span></span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>html</span><span class="token punctuation">&gt;</span></span>
    `</pre>

    The error file is:

    <pre data-title="html" class=" language-html">`{{-- resources\views\errors\list.blade.php --}}
    @if (count($errors) &gt; 0)
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>alert alert-danger<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>ul</span><span class="token punctuation">&gt;</span></span>
                @foreach ($errors-&gt;all() as $error)
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>li</span><span class="token punctuation">&gt;</span></span>{{ $error }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>li</span><span class="token punctuation">&gt;</span></span>
                @endforeach
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>ul</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
    @endif`</pre>

    and the `styles.css` file is simply:

    <pre data-title="css" class=" language-css">`<span class="token selector">p<span class="token class">.teaser</span> </span><span class="token punctuation">{</span>
        <span class="token property">text-indent</span><span class="token punctuation">:</span> <span class="token number">30</span>px<span class="token punctuation">;</span> 
        <span class="token punctuation">}</span>`</pre>

    ### Post Controller

    First, let's create the migration and model files for the `PostController`

    <pre data-title="bash" class=" language-bash">`php artisan make:model Post -m`</pre>

    This command generates a migration file in `app/database/migrations` for generating a new MySQL table named posts in our database and a model file `Post.php`in the `app` directory. Let's edit the migration file to include title and body fields of our post. Add a title and body field so the migration file looks like this: 

    <pre data-title="php" class=" language-php">`<span class="token php language-php"><span class="token delimiter important">&lt;?php</span>
    <span class="token comment">//database\migrations\xxxx_xx_xx_xxxxxx_create_posts_table.php</span>

    <span class="token keyword">use</span> <span class="token package">Illuminate<span class="token punctuation">\</span>Support<span class="token punctuation">\</span>Facades<span class="token punctuation">\</span>Schema</span><span class="token punctuation">;</span>
    <span class="token keyword">use</span> <span class="token package">Illuminate<span class="token punctuation">\</span>Database<span class="token punctuation">\</span>Schema<span class="token punctuation">\</span>Blueprint</span><span class="token punctuation">;</span>
    <span class="token keyword">use</span> <span class="token package">Illuminate<span class="token punctuation">\</span>Database<span class="token punctuation">\</span>Migrations<span class="token punctuation">\</span>Migration</span><span class="token punctuation">;</span>

    <span class="token keyword">class</span> <span class="token class-name">CreatePostsTable</span> <span class="token keyword">extends</span> <span class="token class-name">Migration</span>
    <span class="token punctuation">{</span>
        <span class="token comment">/**
         * Run the migrations.
         *
         * @return void
         */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">up</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
        <span class="token punctuation">{</span>
            Schema<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token string">'posts'</span><span class="token punctuation">,</span> <span class="token keyword">function</span> <span class="token punctuation">(</span>Blueprint <span class="token variable">$table</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
                <span class="token variable">$table</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">increments</span><span class="token punctuation">(</span><span class="token string">'id'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
                <span class="token variable">$table</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">string</span><span class="token punctuation">(</span><span class="token string">'title'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
                <span class="token variable">$table</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">text</span><span class="token punctuation">(</span><span class="token string">'body'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
                <span class="token variable">$table</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">timestamps</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
            <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
         * Reverse the migrations.
         *
         * @return void
         */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">down</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
        <span class="token punctuation">{</span>
            Schema<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">dropIfExists</span><span class="token punctuation">(</span><span class="token string">'posts'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>
    <span class="token punctuation">}</span>
    </span>`</pre>

    After saving the file, run migration again

    <pre data-title="bash" class=" language-bash">`php artisan migrate`</pre>

    You can now check the database for the `post` table and columns. 

    ![](https://cdn.scotch.io/21141/hr25oe8mQCeSVZ3O79PL_acl_db.png)

    Next make the title and body field of the `Post` model [mass assignable](https://laravel.com/docs/5.4/eloquent#mass-assignment)

    <pre data-title="php" class=" language-php">`<span class="token keyword">namespace</span> <span class="token package">App</span><span class="token punctuation">;</span>
    <span class="token keyword">use</span> <span class="token package">Illuminate<span class="token punctuation">\</span>Database<span class="token punctuation">\</span>Eloquent<span class="token punctuation">\</span>Model</span><span class="token punctuation">;</span>

    <span class="token keyword">class</span> <span class="token class-name">Post</span> <span class="token keyword">extends</span> <span class="token class-name">Model</span> <span class="token punctuation">{</span>
        <span class="token keyword">protected</span> <span class="token variable">$fillable</span> <span class="token operator">=</span> <span class="token punctuation">[</span>
            <span class="token string">'title'</span><span class="token punctuation">,</span> <span class="token string">'body'</span>
        <span class="token punctuation">]</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>`</pre>

    Now let's generate our resource controller.

    <pre data-title="bash" class=" language-bash">`php artisan make:controller PostController --resource`</pre>

    This will create our controller with all the stub methods needed. Edit this file to look like this

    <pre data-title="php" class=" language-php">`<span class="token php language-php"><span class="token delimiter important">&lt;?php</span>
    <span class="token comment">// app/Http/Controllers/PostController.php</span>

    <span class="token keyword">namespace</span> <span class="token package">App<span class="token punctuation">\</span>Http<span class="token punctuation">\</span>Controllers</span><span class="token punctuation">;</span>

    <span class="token keyword">use</span> <span class="token package">Illuminate<span class="token punctuation">\</span>Http<span class="token punctuation">\</span>Request</span><span class="token punctuation">;</span>

    <span class="token keyword">use</span> <span class="token package">App<span class="token punctuation">\</span>Post</span><span class="token punctuation">;</span>
    <span class="token keyword">use</span> <span class="token package">Auth</span><span class="token punctuation">;</span>
    <span class="token keyword">use</span> <span class="token package">Session</span><span class="token punctuation">;</span>

    <span class="token keyword">class</span> <span class="token class-name">PostController</span> <span class="token keyword">extends</span> <span class="token class-name">Controller</span> <span class="token punctuation">{</span>

        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">__construct</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">middleware</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token string">'auth'</span><span class="token punctuation">,</span> <span class="token string">'clearance'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">except</span><span class="token punctuation">(</span><span class="token string">'index'</span><span class="token punctuation">,</span> <span class="token string">'show'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
         * Display a listing of the resource.
         *
         * @return \Illuminate\Http\Response
         */</span>

        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">index</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$posts</span> <span class="token operator">=</span> Post<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">orderby</span><span class="token punctuation">(</span><span class="token string">'id'</span><span class="token punctuation">,</span> <span class="token string">'desc'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">paginate</span><span class="token punctuation">(</span><span class="token number">5</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//show only 5 items at a time in descending order</span>

            <span class="token keyword">return</span> <span class="token function">view</span><span class="token punctuation">(</span><span class="token string">'posts.index'</span><span class="token punctuation">,</span> <span class="token function">compact</span><span class="token punctuation">(</span><span class="token string">'posts'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
         * Show the form for creating a new resource.
         *
         * @return \Illuminate\Http\Response
         */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token keyword">return</span> <span class="token function">view</span><span class="token punctuation">(</span><span class="token string">'posts.create'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
         * Store a newly created resource in storage.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">store</span><span class="token punctuation">(</span>Request <span class="token variable">$request</span><span class="token punctuation">)</span> <span class="token punctuation">{</span> 

        <span class="token comment">//Validating title and body field</span>
            <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">validate</span><span class="token punctuation">(</span><span class="token variable">$request</span><span class="token punctuation">,</span> <span class="token punctuation">[</span>
                <span class="token string">'title'</span><span class="token operator">=</span><span class="token operator">&gt;</span><span class="token string">'required|max:100'</span><span class="token punctuation">,</span>
                <span class="token string">'body'</span> <span class="token operator">=</span><span class="token operator">&gt;</span><span class="token string">'required'</span><span class="token punctuation">,</span>
                <span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

            <span class="token variable">$title</span> <span class="token operator">=</span> <span class="token variable">$request</span><span class="token punctuation">[</span><span class="token string">'title'</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
            <span class="token variable">$body</span> <span class="token operator">=</span> <span class="token variable">$request</span><span class="token punctuation">[</span><span class="token string">'body'</span><span class="token punctuation">]</span><span class="token punctuation">;</span>

            <span class="token variable">$post</span> <span class="token operator">=</span> Post<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token variable">$request</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">only</span><span class="token punctuation">(</span><span class="token string">'title'</span><span class="token punctuation">,</span> <span class="token string">'body'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

        <span class="token comment">//Display a successful message upon save</span>
            <span class="token keyword">return</span> <span class="token function">redirect</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">route</span><span class="token punctuation">(</span><span class="token string">'posts.index'</span><span class="token punctuation">)</span>
                <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">with</span><span class="token punctuation">(</span><span class="token string">'flash_message'</span><span class="token punctuation">,</span> <span class="token string">'Article,
                 '</span><span class="token punctuation">.</span> <span class="token variable">$post</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">title</span><span class="token punctuation">.</span><span class="token string">' created'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
         * Display the specified resource.
         *
         * @param  int  $id
         * @return \Illuminate\Http\Response
         */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">show</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$post</span> <span class="token operator">=</span> Post<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">findOrFail</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//Find post of id = $id</span>

            <span class="token keyword">return</span> view <span class="token punctuation">(</span><span class="token string">'posts.show'</span><span class="token punctuation">,</span> <span class="token function">compact</span><span class="token punctuation">(</span><span class="token string">'post'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
         * Show the form for editing the specified resource.
         *
         * @param  int  $id
         * @return \Illuminate\Http\Response
         */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">edit</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$post</span> <span class="token operator">=</span> Post<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">findOrFail</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

            <span class="token keyword">return</span> <span class="token function">view</span><span class="token punctuation">(</span><span class="token string">'posts.edit'</span><span class="token punctuation">,</span> <span class="token function">compact</span><span class="token punctuation">(</span><span class="token string">'post'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
         * Update the specified resource in storage.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  int  $id
         * @return \Illuminate\Http\Response
         */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">update</span><span class="token punctuation">(</span>Request <span class="token variable">$request</span><span class="token punctuation">,</span> <span class="token variable">$id</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">validate</span><span class="token punctuation">(</span><span class="token variable">$request</span><span class="token punctuation">,</span> <span class="token punctuation">[</span>
                <span class="token string">'title'</span><span class="token operator">=</span><span class="token operator">&gt;</span><span class="token string">'required|max:100'</span><span class="token punctuation">,</span>
                <span class="token string">'body'</span><span class="token operator">=</span><span class="token operator">&gt;</span><span class="token string">'required'</span><span class="token punctuation">,</span>
            <span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

            <span class="token variable">$post</span> <span class="token operator">=</span> Post<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">findOrFail</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
            <span class="token variable">$post</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">title</span> <span class="token operator">=</span> <span class="token variable">$request</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">input</span><span class="token punctuation">(</span><span class="token string">'title'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
            <span class="token variable">$post</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">body</span> <span class="token operator">=</span> <span class="token variable">$request</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">input</span><span class="token punctuation">(</span><span class="token string">'body'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
            <span class="token variable">$post</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">save</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

            <span class="token keyword">return</span> <span class="token function">redirect</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">route</span><span class="token punctuation">(</span><span class="token string">'posts.show'</span><span class="token punctuation">,</span> 
                <span class="token variable">$post</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">id</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">with</span><span class="token punctuation">(</span><span class="token string">'flash_message'</span><span class="token punctuation">,</span> 
                <span class="token string">'Article, '</span><span class="token punctuation">.</span> <span class="token variable">$post</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">title</span><span class="token punctuation">.</span><span class="token string">' updated'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

        <span class="token punctuation">}</span>

        <span class="token comment">/**
         * Remove the specified resource from storage.
         *
         * @param  int  $id
         * @return \Illuminate\Http\Response
         */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">destroy</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$post</span> <span class="token operator">=</span> Post<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">findOrFail</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
            <span class="token variable">$post</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">delete</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

            <span class="token keyword">return</span> <span class="token function">redirect</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">route</span><span class="token punctuation">(</span><span class="token string">'posts.index'</span><span class="token punctuation">)</span>
                <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">with</span><span class="token punctuation">(</span><span class="token string">'flash_message'</span><span class="token punctuation">,</span>
                 <span class="token string">'Article successfully deleted'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

        <span class="token punctuation">}</span>
    <span class="token punctuation">}</span></span>`</pre>

    Here the `Post` class was imported from our model and the `Auth` class which was generated with the `make:auth` command earlier. These were imported so that you would be able to make Eloquent queries on the `Post` table and so as to be able to have access to authentication information of our users. In the constructor two middlewares were called, one is `auth` which restricts access to the `PostController` methods to authenticated users the other is a custom middleware is yet to be created. This would be responsible for our Permissions and Roles system. Next, `index` and `show` are passed into the except method to allow all users to be able to view posts.

    The `index()` method lists all the available posts. It queries the `post` table for all posts and passes this information to the view. `Paginate()` allows us to limit the number of posts in a page, in this case five. 

    The `create()` method simply returns the `posts/create` view which would contain a form for creating new posts. The `store()` method saves the information input from the `posts/create` view. The information is first validated and after it is saved, a flash message is passed to the view `posts/index`.

    Our `show()` method of the `PostController` allows us to display a single post. This method takes the `post` id as an argument and passes it to the method `Post::find()`. The result of the query is then sent to our `posts/show` view.

    The `edit()` method, similar to the `create()` method simply returns the `posts/edit` view which would contain a form for creating editing posts. The `update()` method takes the information from the `posts/edit` view and updates the record. The `destroy()` method let's us delete a post. 

    Now that you have the `PostController` you need to set up the routes. Edit your `app/routes/web.php` file to look like this:

    <pre data-title="php" class=" language-php">`<span class="token php language-php"><span class="token delimiter important">&lt;?php</span>

    Route<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'/'</span><span class="token punctuation">,</span> <span class="token keyword">function</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token keyword">return</span> <span class="token function">view</span><span class="token punctuation">(</span><span class="token string">'welcome'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

    Auth<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">routes</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

    Route<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'/'</span><span class="token punctuation">,</span> <span class="token string">'PostController@index'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">name</span><span class="token punctuation">(</span><span class="token string">'home'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

    Route<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">resource</span><span class="token punctuation">(</span><span class="token string">'users'</span><span class="token punctuation">,</span> <span class="token string">'UserController'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

    Route<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">resource</span><span class="token punctuation">(</span><span class="token string">'roles'</span><span class="token punctuation">,</span> <span class="token string">'RoleController'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

    Route<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">resource</span><span class="token punctuation">(</span><span class="token string">'permissions'</span><span class="token punctuation">,</span> <span class="token string">'PermissionController'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

    Route<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">resource</span><span class="token punctuation">(</span><span class="token string">'posts'</span><span class="token punctuation">,</span> <span class="token string">'PostController'</span><span class="token punctuation">)</span><span class="token punctuation">;</span></span>`</pre>

    The `/` route is the route to our home page, here it was renamed to `home` The `Auth` route was generated when you ran the `make:auth` command. It handles authentication related routes. The other four routes are for resources that would be created later.

    ### Post Views

    Only four views are needed for our `PostController`. Create the files `\resources\views\posts\index.blade.php`, `\resources\views\posts\create.blade.php`, `\resources\views\posts\show.blade.php`, `\resources\views\posts\edit.blade.php`

    Edit the `index.blade.php`file to look like this

    <pre data-title="html" class=" language-html">`@extends('layouts.app')
    @section('content')
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>container<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>row<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>col-md-10 col-md-offset-1<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>panel panel-default<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>panel-heading<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>h3</span><span class="token punctuation">&gt;</span></span>Posts<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>h3</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>panel-heading<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>Page {{ $posts-&gt;currentPage() }} of {{ $posts-&gt;lastPage() }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
                        @foreach ($posts as $post)
                            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>panel-body<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>li</span> <span class="token attr-name">style</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>list-style-type:disc<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ route(<span class="token punctuation">'</span>posts.show<span class="token punctuation">'</span>, $post-&gt;id ) }}<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>b</span><span class="token punctuation">&gt;</span></span>{{ $post-&gt;title }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>b</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>
                                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>p</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>teaser<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                                           {{  str_limit($post-&gt;body, 100) }} {{-- Limit teaser to 100 characters --}}
                                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>p</span><span class="token punctuation">&gt;</span></span>
                                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span>
                                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>li</span><span class="token punctuation">&gt;</span></span>
                            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
                        @endforeach
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>text-center<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                            {!! $posts-&gt;links() !!}
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
    @endsection`</pre>

    Notice that this file extends `views\layouts\app.php` file, which was generated earlier by the `make:auth` command. 

    The `create.blade.php` file looks like this

    <pre data-title="html" class=" language-html">`@extends('layouts.app')

    @section('title', '| Create New Post')

    @section('content')
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>row<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>col-md-8 col-md-offset-2<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>

            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>h1</span><span class="token punctuation">&gt;</span></span>Create New Post<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>h1</span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>hr</span><span class="token punctuation">&gt;</span></span>

        {{-- Using the Laravel HTML Form Collective to create our form --}}
            {{ Form::open(array('route' =&gt; 'posts.store')) }}

            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>form-group<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                {{ Form::label('title', 'Title') }}
                {{ Form::text('title', null, array('class' =&gt; 'form-control')) }}
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>

                {{ Form::label('body', 'Post Body') }}
                {{ Form::textarea('body', null, array('class' =&gt; 'form-control')) }}
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>

                {{ Form::submit('Create Post', array('class' =&gt; 'btn btn-success btn-lg btn-block')) }}
                {{ Form::close() }}
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

    @endsection`</pre>

    The `show` view looks like this:

    <pre data-title="html" class=" language-html">`@extends('layouts.app')

    @section('title', '| View Post')

    @section('content')

    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>container<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>h1</span><span class="token punctuation">&gt;</span></span>{{ $post-&gt;title }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>h1</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>hr</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>p</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>lead<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>{{ $post-&gt;body }} <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>p</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>hr</span><span class="token punctuation">&gt;</span></span>
        {!! Form::open(['method' =&gt; 'DELETE', 'route' =&gt; ['posts.destroy', $post-&gt;id] ]) !!}
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ url()-&gt;previous() }}<span class="token punctuation">"</span></span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>btn btn-primary<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>Back<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span>
        @can('Edit Post')
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ route(<span class="token punctuation">'</span>posts.edit<span class="token punctuation">'</span>, $post-&gt;id) }}<span class="token punctuation">"</span></span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>btn btn-info<span class="token punctuation">"</span></span> <span class="token attr-name">role</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>button<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>Edit<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span>
        @endcan
        @can('Delete Post')
        {!! Form::submit('Delete', ['class' =&gt; 'btn btn-danger']) !!}
        @endcan
        {!! Form::close() !!}

    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

    @endsection`</pre>

    Here the `can` directive checks if a user has the permission to Edit or Delete Posts, if so the Edit and Delete button will be displayed. If the user does not have these permissions, only the Back button would be displayed.

    The `edit` view just displays a edit form that will be used to update records:

    <pre data-title="html" class=" language-html">`@extends('layouts.app')

    @section('title', '| Edit Post')

    @section('content')
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>row<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>col-md-8 col-md-offset-2<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>

            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>h1</span><span class="token punctuation">&gt;</span></span>Edit Post<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>h1</span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>hr</span><span class="token punctuation">&gt;</span></span>
                {{ Form::model($post, array('route' =&gt; array('posts.update', $post-&gt;id), 'method' =&gt; 'PUT')) }}
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>form-group<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                {{ Form::label('title', 'Title') }}
                {{ Form::text('title', null, array('class' =&gt; 'form-control')) }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>

                {{ Form::label('body', 'Post Body') }}
                {{ Form::textarea('body', null, array('class' =&gt; 'form-control')) }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>

                {{ Form::submit('Save', array('class' =&gt; 'btn btn-primary')) }}

                {{ Form::close() }}
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

    @endsection`</pre>

    If you visit the home page you would see this
    ![](https://cdn.scotch.io/21141/8f91qIfOQNC121PjnYAG_home.png)

    ### User Controller

    The `UserController` will handle displaying all users, creating of new users, editing users, assigning roles to users and deleting users. As before generate the controller by running

    <pre data-title="bash" class=" language-bash">`php artisan make:controller UserController --resource`</pre>

    Then replace the content of this file with:

    <pre data-title="php" class=" language-php">`<span class="token php language-php"><span class="token delimiter important">&lt;?php</span>

    <span class="token keyword">namespace</span> <span class="token package">App<span class="token punctuation">\</span>Http<span class="token punctuation">\</span>Controllers</span><span class="token punctuation">;</span>

    <span class="token keyword">use</span> <span class="token package">Illuminate<span class="token punctuation">\</span>Http<span class="token punctuation">\</span>Request</span><span class="token punctuation">;</span>

    <span class="token keyword">use</span> <span class="token package">App<span class="token punctuation">\</span>User</span><span class="token punctuation">;</span>
    <span class="token keyword">use</span> <span class="token package">Auth</span><span class="token punctuation">;</span>

    <span class="token comment">//Importing laravel-permission models</span>
    <span class="token keyword">use</span> <span class="token package">Spatie<span class="token punctuation">\</span>Permission<span class="token punctuation">\</span>Models<span class="token punctuation">\</span>Role</span><span class="token punctuation">;</span>
    <span class="token keyword">use</span> <span class="token package">Spatie<span class="token punctuation">\</span>Permission<span class="token punctuation">\</span>Models<span class="token punctuation">\</span>Permission</span><span class="token punctuation">;</span>

    <span class="token comment">//Enables us to output flash messaging</span>
    <span class="token keyword">use</span> <span class="token package">Session</span><span class="token punctuation">;</span>

    <span class="token keyword">class</span> <span class="token class-name">UserController</span> <span class="token keyword">extends</span> <span class="token class-name">Controller</span> <span class="token punctuation">{</span>

        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">__construct</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">middleware</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token string">'auth'</span><span class="token punctuation">,</span> <span class="token string">'isAdmin'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//isAdmin middleware lets only users with a //specific permission permission to access these resources</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
        * Display a listing of the resource.
        *
        * @return \Illuminate\Http\Response
        */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">index</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token comment">//Get all users and pass it to the view</span>
            <span class="token variable">$users</span> <span class="token operator">=</span> User<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">all</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> 
            <span class="token keyword">return</span> <span class="token function">view</span><span class="token punctuation">(</span><span class="token string">'users.index'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">with</span><span class="token punctuation">(</span><span class="token string">'users'</span><span class="token punctuation">,</span> <span class="token variable">$users</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
        * Show the form for creating a new resource.
        *
        * @return \Illuminate\Http\Response
        */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token comment">//Get all roles and pass it to the view</span>
            <span class="token variable">$roles</span> <span class="token operator">=</span> Role<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
            <span class="token keyword">return</span> <span class="token function">view</span><span class="token punctuation">(</span><span class="token string">'users.create'</span><span class="token punctuation">,</span> <span class="token punctuation">[</span><span class="token string">'roles'</span><span class="token operator">=</span><span class="token operator">&gt;</span><span class="token variable">$roles</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
        * Store a newly created resource in storage.
        *
        * @param  \Illuminate\Http\Request  $request
        * @return \Illuminate\Http\Response
        */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">store</span><span class="token punctuation">(</span>Request <span class="token variable">$request</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token comment">//Validate name, email and password fields</span>
            <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">validate</span><span class="token punctuation">(</span><span class="token variable">$request</span><span class="token punctuation">,</span> <span class="token punctuation">[</span>
                <span class="token string">'name'</span><span class="token operator">=</span><span class="token operator">&gt;</span><span class="token string">'required|max:120'</span><span class="token punctuation">,</span>
                <span class="token string">'email'</span><span class="token operator">=</span><span class="token operator">&gt;</span><span class="token string">'required|email|unique:users'</span><span class="token punctuation">,</span>
                <span class="token string">'password'</span><span class="token operator">=</span><span class="token operator">&gt;</span><span class="token string">'required|min:6|confirmed'</span>
            <span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

            <span class="token variable">$user</span> <span class="token operator">=</span> User<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token variable">$request</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">only</span><span class="token punctuation">(</span><span class="token string">'email'</span><span class="token punctuation">,</span> <span class="token string">'name'</span><span class="token punctuation">,</span> <span class="token string">'password'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//Retrieving only the email and password data</span>

            <span class="token variable">$roles</span> <span class="token operator">=</span> <span class="token variable">$request</span><span class="token punctuation">[</span><span class="token string">'roles'</span><span class="token punctuation">]</span><span class="token punctuation">;</span> <span class="token comment">//Retrieving the roles field</span>
        <span class="token comment">//Checking if a role was selected</span>
            <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token function">isset</span><span class="token punctuation">(</span><span class="token variable">$roles</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>

                <span class="token keyword">foreach</span> <span class="token punctuation">(</span><span class="token variable">$roles</span> <span class="token keyword">as</span> <span class="token variable">$role</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
                <span class="token variable">$role_r</span> <span class="token operator">=</span> Role<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">where</span><span class="token punctuation">(</span><span class="token string">'id'</span><span class="token punctuation">,</span> <span class="token string">'='</span><span class="token punctuation">,</span> <span class="token variable">$role</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">firstOrFail</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>            
                <span class="token variable">$user</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">assignRole</span><span class="token punctuation">(</span><span class="token variable">$role_r</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//Assigning role to user</span>
                <span class="token punctuation">}</span>
            <span class="token punctuation">}</span>        
        <span class="token comment">//Redirect to the users.index view and display message</span>
            <span class="token keyword">return</span> <span class="token function">redirect</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">route</span><span class="token punctuation">(</span><span class="token string">'users.index'</span><span class="token punctuation">)</span>
                <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">with</span><span class="token punctuation">(</span><span class="token string">'flash_message'</span><span class="token punctuation">,</span>
                 <span class="token string">'User successfully added.'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
        * Display the specified resource.
        *
        * @param  int  $id
        * @return \Illuminate\Http\Response
        */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">show</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token keyword">return</span> <span class="token function">redirect</span><span class="token punctuation">(</span><span class="token string">'users'</span><span class="token punctuation">)</span><span class="token punctuation">;</span> 
        <span class="token punctuation">}</span>

        <span class="token comment">/**
        * Show the form for editing the specified resource.
        *
        * @param  int  $id
        * @return \Illuminate\Http\Response
        */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">edit</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$user</span> <span class="token operator">=</span> User<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">findOrFail</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//Get user with specified id</span>
            <span class="token variable">$roles</span> <span class="token operator">=</span> Role<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//Get all roles</span>

            <span class="token keyword">return</span> <span class="token function">view</span><span class="token punctuation">(</span><span class="token string">'users.edit'</span><span class="token punctuation">,</span> <span class="token function">compact</span><span class="token punctuation">(</span><span class="token string">'user'</span><span class="token punctuation">,</span> <span class="token string">'roles'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//pass user and roles data to view</span>

        <span class="token punctuation">}</span>

        <span class="token comment">/**
        * Update the specified resource in storage.
        *
        * @param  \Illuminate\Http\Request  $request
        * @param  int  $id
        * @return \Illuminate\Http\Response
        */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">update</span><span class="token punctuation">(</span>Request <span class="token variable">$request</span><span class="token punctuation">,</span> <span class="token variable">$id</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$user</span> <span class="token operator">=</span> User<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">findOrFail</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//Get role specified by id</span>

        <span class="token comment">//Validate name, email and password fields  </span>
            <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">validate</span><span class="token punctuation">(</span><span class="token variable">$request</span><span class="token punctuation">,</span> <span class="token punctuation">[</span>
                <span class="token string">'name'</span><span class="token operator">=</span><span class="token operator">&gt;</span><span class="token string">'required|max:120'</span><span class="token punctuation">,</span>
                <span class="token string">'email'</span><span class="token operator">=</span><span class="token operator">&gt;</span><span class="token string">'required|email|unique:users,email,'</span><span class="token punctuation">.</span><span class="token variable">$id</span><span class="token punctuation">,</span>
                <span class="token string">'password'</span><span class="token operator">=</span><span class="token operator">&gt;</span><span class="token string">'required|min:6|confirmed'</span>
            <span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
            <span class="token variable">$input</span> <span class="token operator">=</span> <span class="token variable">$request</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">only</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token string">'name'</span><span class="token punctuation">,</span> <span class="token string">'email'</span><span class="token punctuation">,</span> <span class="token string">'password'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//Retreive the name, email and password fields</span>
            <span class="token variable">$roles</span> <span class="token operator">=</span> <span class="token variable">$request</span><span class="token punctuation">[</span><span class="token string">'roles'</span><span class="token punctuation">]</span><span class="token punctuation">;</span> <span class="token comment">//Retreive all roles</span>
            <span class="token variable">$user</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">fill</span><span class="token punctuation">(</span><span class="token variable">$input</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">save</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

            <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token function">isset</span><span class="token punctuation">(</span><span class="token variable">$roles</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>        
                <span class="token variable">$user</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">roles</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">sync</span><span class="token punctuation">(</span><span class="token variable">$roles</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  <span class="token comment">//If one or more role is selected associate user to roles          </span>
            <span class="token punctuation">}</span>        
            <span class="token keyword">else</span> <span class="token punctuation">{</span>
                <span class="token variable">$user</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">roles</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">detach</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//If no role is selected remove exisiting role associated to a user</span>
            <span class="token punctuation">}</span>
            <span class="token keyword">return</span> <span class="token function">redirect</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">route</span><span class="token punctuation">(</span><span class="token string">'users.index'</span><span class="token punctuation">)</span>
                <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">with</span><span class="token punctuation">(</span><span class="token string">'flash_message'</span><span class="token punctuation">,</span>
                 <span class="token string">'User successfully edited.'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
        * Remove the specified resource from storage.
        *
        * @param  int  $id
        * @return \Illuminate\Http\Response
        */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">destroy</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token comment">//Find a user with a given id and delete</span>
            <span class="token variable">$user</span> <span class="token operator">=</span> User<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">findOrFail</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span><span class="token punctuation">;</span> 
            <span class="token variable">$user</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">delete</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

            <span class="token keyword">return</span> <span class="token function">redirect</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">route</span><span class="token punctuation">(</span><span class="token string">'users.index'</span><span class="token punctuation">)</span>
                <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">with</span><span class="token punctuation">(</span><span class="token string">'flash_message'</span><span class="token punctuation">,</span>
                 <span class="token string">'User successfully deleted.'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>
    <span class="token punctuation">}</span></span>`</pre>

    Here the `User` class, the `Role` class, the `Permission` class and the `Auth` class are imported. In the constructor the `auth` middleware is called to make sure only authenticated users have access to the User resource. A custom middleware `isAdmin` is also called. This checks if the authenticated user has administrator privileges. This middleware will be created later.

    The `index()` method gets all users from the Users table and passes it to the index view which will display all users in a table. The `create()` method first gets all the Roles from the Roles table and passes it to the create view. This is so that Roles can be added when creating a User.

    The `store()` method saves the input from the create view, after validating the input, looping through the Roles that was passed in the form and assigning these Roles to the User. The `show()`method just redirects back to the users page as for this demonstration, we wont need to show each user individually.

    The `edit()` method gets the user corresponding to the id passed, then gets all `roles` and passes it to the edit view. The `update()` method validates data from the edit view and saves the updated name and password fields. It gets all `roles` from the `roles` table and while looping through them, removes any `role` assign to the user. It then takes the `role` data inputted from the form, matches them with the values in the databases and assigns these `roles` to the user.

    The `destroy()` method allows us to delete a `user` along with it's corresponding `role`.

    ### User Views

    Three views are needed here: `index`, `create` and `edit` views. The `index` view would contain a table that lists all our `users` and their `roles`.

    <pre data-title="html" class=" language-html">`{{-- \resources\views\users\index.blade.php --}}
    @extends('layouts.app')

    @section('title', '| Users')

    @section('content')

    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>col-lg-10 col-lg-offset-1<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>h1</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>i</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>fa fa-users<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>i</span><span class="token punctuation">&gt;</span></span> User Administration <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ route(<span class="token punctuation">'</span>roles.index<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>btn btn-default pull-right<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>Roles<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ route(<span class="token punctuation">'</span>permissions.index<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>btn btn-default pull-right<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>Permissions<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>h1</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>hr</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>table-responsive<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>table</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>table table-bordered table-striped<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>

                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>thead</span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>tr</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>th</span><span class="token punctuation">&gt;</span></span>Name<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>th</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>th</span><span class="token punctuation">&gt;</span></span>Email<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>th</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>th</span><span class="token punctuation">&gt;</span></span>Date/Time Added<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>th</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>th</span><span class="token punctuation">&gt;</span></span>User Roles<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>th</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>th</span><span class="token punctuation">&gt;</span></span>Operations<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>th</span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>tr</span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>thead</span><span class="token punctuation">&gt;</span></span>

                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>tbody</span><span class="token punctuation">&gt;</span></span>
                    @foreach ($users as $user)
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>tr</span><span class="token punctuation">&gt;</span></span>

                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>td</span><span class="token punctuation">&gt;</span></span>{{ $user-&gt;name }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>td</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>td</span><span class="token punctuation">&gt;</span></span>{{ $user-&gt;email }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>td</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>td</span><span class="token punctuation">&gt;</span></span>{{ $user-&gt;created_at-&gt;format('F d, Y h:ia') }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>td</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>td</span><span class="token punctuation">&gt;</span></span>{{  $user-&gt;roles()-&gt;pluck('name')-&gt;implode(' ') }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>td</span><span class="token punctuation">&gt;</span></span>{{-- Retrieve array of roles associated to a user and convert to string --}}
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>td</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ route(<span class="token punctuation">'</span>users.edit<span class="token punctuation">'</span>, $user-&gt;id) }}<span class="token punctuation">"</span></span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>btn btn-info pull-left<span class="token punctuation">"</span></span> <span class="token attr-name">style</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>margin-right: 3px;<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>Edit<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span>

                        {!! Form::open(['method' =&gt; 'DELETE', 'route' =&gt; ['users.destroy', $user-&gt;id] ]) !!}
                        {!! Form::submit('Delete', ['class' =&gt; 'btn btn-danger']) !!}
                        {!! Form::close() !!}

                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>td</span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>tr</span><span class="token punctuation">&gt;</span></span>
                    @endforeach
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>tbody</span><span class="token punctuation">&gt;</span></span>

            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>table</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ route(<span class="token punctuation">'</span>users.create<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>btn btn-success<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>Add User<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span>

    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

    @endsection`</pre>

    The `create` view is just a form that allows us to create new `users` and assign `roles` to them.

    <pre data-title="html" class=" language-html">`{{-- \resources\views\users\create.blade.php --}}
    @extends('layouts.app')

    @section('title', '| Add User')

    @section('content')

    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">'</span>col-lg-4 col-lg-offset-4<span class="token punctuation">'</span></span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>h1</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>i</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">'</span>fa fa-user-plus<span class="token punctuation">'</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>i</span><span class="token punctuation">&gt;</span></span> Add User<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>h1</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>hr</span><span class="token punctuation">&gt;</span></span>

        {{ Form::open(array('url' =&gt; 'users')) }}

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>form-group<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            {{ Form::label('name', 'Name') }}
            {{ Form::text('name', '', array('class' =&gt; 'form-control')) }}
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>form-group<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            {{ Form::label('email', 'Email') }}
            {{ Form::email('email', '', array('class' =&gt; 'form-control')) }}
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">'</span>form-group<span class="token punctuation">'</span></span><span class="token punctuation">&gt;</span></span>
            @foreach ($roles as $role)
                {{ Form::checkbox('roles[]',  $role-&gt;id ) }}
                {{ Form::label($role-&gt;name, ucfirst($role-&gt;name)) }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>

            @endforeach
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>form-group<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            {{ Form::label('password', 'Password') }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>
            {{ Form::password('password', array('class' =&gt; 'form-control')) }}

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>form-group<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            {{ Form::label('password', 'Confirm Password') }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>
            {{ Form::password('password_confirmation', array('class' =&gt; 'form-control')) }}

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

        {{ Form::submit('Add', array('class' =&gt; 'btn btn-primary')) }}

        {{ Form::close() }}

    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

    @endsection`</pre>

    The `edit` view is a form that allows us to edit `users` and their `roles`. Using Laravel's form model binding the form is automatically populated with the previous values.

    <pre data-title="html" class=" language-html">`{{-- \resources\views\users\edit.blade.php --}}

    @extends('layouts.app')

    @section('title', '| Edit User')

    @section('content')

    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">'</span>col-lg-4 col-lg-offset-4<span class="token punctuation">'</span></span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>h1</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>i</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">'</span>fa fa-user-plus<span class="token punctuation">'</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>i</span><span class="token punctuation">&gt;</span></span> Edit {{$user-&gt;name}}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>h1</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>hr</span><span class="token punctuation">&gt;</span></span>

        {{ Form::model($user, array('route' =&gt; array('users.update', $user-&gt;id), 'method' =&gt; 'PUT')) }}{{-- Form model binding to automatically populate our fields with user data --}}

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>form-group<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            {{ Form::label('name', 'Name') }}
            {{ Form::text('name', null, array('class' =&gt; 'form-control')) }}
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>form-group<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            {{ Form::label('email', 'Email') }}
            {{ Form::email('email', null, array('class' =&gt; 'form-control')) }}
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>h5</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>b</span><span class="token punctuation">&gt;</span></span>Give Role<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>b</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>h5</span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">'</span>form-group<span class="token punctuation">'</span></span><span class="token punctuation">&gt;</span></span>
            @foreach ($roles as $role)
                {{ Form::checkbox('roles[]',  $role-&gt;id, $user-&gt;roles ) }}
                {{ Form::label($role-&gt;name, ucfirst($role-&gt;name)) }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>

            @endforeach
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>form-group<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            {{ Form::label('password', 'Password') }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>
            {{ Form::password('password', array('class' =&gt; 'form-control')) }}

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>form-group<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            {{ Form::label('password', 'Confirm Password') }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>
            {{ Form::password('password_confirmation', array('class' =&gt; 'form-control')) }}

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

        {{ Form::submit('Add', array('class' =&gt; 'btn btn-primary')) }}

        {{ Form::close() }}

    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

    @endsection`</pre>

    ### Permission Controller

    Now let's tackle the `PermissionController`Create the file and paste the following code:

    <pre data-title="php" class=" language-php">`<span class="token php language-php"><span class="token delimiter important">&lt;?php</span>

    <span class="token keyword">namespace</span> <span class="token package">App<span class="token punctuation">\</span>Http<span class="token punctuation">\</span>Controllers</span><span class="token punctuation">;</span>

    <span class="token keyword">use</span> <span class="token package">Illuminate<span class="token punctuation">\</span>Http<span class="token punctuation">\</span>Request</span><span class="token punctuation">;</span>

    <span class="token keyword">use</span> <span class="token package">Auth</span><span class="token punctuation">;</span>

    <span class="token comment">//Importing laravel-permission models</span>
    <span class="token keyword">use</span> <span class="token package">Spatie<span class="token punctuation">\</span>Permission<span class="token punctuation">\</span>Models<span class="token punctuation">\</span>Role</span><span class="token punctuation">;</span>
    <span class="token keyword">use</span> <span class="token package">Spatie<span class="token punctuation">\</span>Permission<span class="token punctuation">\</span>Models<span class="token punctuation">\</span>Permission</span><span class="token punctuation">;</span>

    <span class="token keyword">use</span> <span class="token package">Session</span><span class="token punctuation">;</span>

    <span class="token keyword">class</span> <span class="token class-name">PermissionController</span> <span class="token keyword">extends</span> <span class="token class-name">Controller</span> <span class="token punctuation">{</span>

        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">__construct</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">middleware</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token string">'auth'</span><span class="token punctuation">,</span> <span class="token string">'isAdmin'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//isAdmin middleware lets only users with a //specific permission permission to access these resources</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
        * Display a listing of the resource.
        *
        * @return \Illuminate\Http\Response
        */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">index</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$permissions</span> <span class="token operator">=</span> Permission<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">all</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//Get all permissions</span>

            <span class="token keyword">return</span> <span class="token function">view</span><span class="token punctuation">(</span><span class="token string">'permissions.index'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">with</span><span class="token punctuation">(</span><span class="token string">'permissions'</span><span class="token punctuation">,</span> <span class="token variable">$permissions</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
        * Show the form for creating a new resource.
        *
        * @return \Illuminate\Http\Response
        */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$roles</span> <span class="token operator">=</span> Role<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//Get all roles</span>

            <span class="token keyword">return</span> <span class="token function">view</span><span class="token punctuation">(</span><span class="token string">'permissions.create'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">with</span><span class="token punctuation">(</span><span class="token string">'roles'</span><span class="token punctuation">,</span> <span class="token variable">$roles</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
        * Store a newly created resource in storage.
        *
        * @param  \Illuminate\Http\Request  $request
        * @return \Illuminate\Http\Response
        */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">store</span><span class="token punctuation">(</span>Request <span class="token variable">$request</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">validate</span><span class="token punctuation">(</span><span class="token variable">$request</span><span class="token punctuation">,</span> <span class="token punctuation">[</span>
                <span class="token string">'name'</span><span class="token operator">=</span><span class="token operator">&gt;</span><span class="token string">'required|max:40'</span><span class="token punctuation">,</span>
            <span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

            <span class="token variable">$name</span> <span class="token operator">=</span> <span class="token variable">$request</span><span class="token punctuation">[</span><span class="token string">'name'</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
            <span class="token variable">$permission</span> <span class="token operator">=</span> <span class="token keyword">new</span> <span class="token class-name">Permission</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
            <span class="token variable">$permission</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">name</span> <span class="token operator">=</span> <span class="token variable">$name</span><span class="token punctuation">;</span>

            <span class="token variable">$roles</span> <span class="token operator">=</span> <span class="token variable">$request</span><span class="token punctuation">[</span><span class="token string">'roles'</span><span class="token punctuation">]</span><span class="token punctuation">;</span>

            <span class="token variable">$permission</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">save</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

            <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span><span class="token function">empty</span><span class="token punctuation">(</span><span class="token variable">$request</span><span class="token punctuation">[</span><span class="token string">'roles'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span> <span class="token comment">//If one or more role is selected</span>
                <span class="token keyword">foreach</span> <span class="token punctuation">(</span><span class="token variable">$roles</span> <span class="token keyword">as</span> <span class="token variable">$role</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
                    <span class="token variable">$r</span> <span class="token operator">=</span> Role<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">where</span><span class="token punctuation">(</span><span class="token string">'id'</span><span class="token punctuation">,</span> <span class="token string">'='</span><span class="token punctuation">,</span> <span class="token variable">$role</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">firstOrFail</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//Match input role to db record</span>

                    <span class="token variable">$permission</span> <span class="token operator">=</span> Permission<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">where</span><span class="token punctuation">(</span><span class="token string">'name'</span><span class="token punctuation">,</span> <span class="token string">'='</span><span class="token punctuation">,</span> <span class="token variable">$name</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">first</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//Match input //permission to db record</span>
                    <span class="token variable">$r</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">givePermissionTo</span><span class="token punctuation">(</span><span class="token variable">$permission</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
                <span class="token punctuation">}</span>
            <span class="token punctuation">}</span>

            <span class="token keyword">return</span> <span class="token function">redirect</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">route</span><span class="token punctuation">(</span><span class="token string">'permissions.index'</span><span class="token punctuation">)</span>
                <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">with</span><span class="token punctuation">(</span><span class="token string">'flash_message'</span><span class="token punctuation">,</span>
                 <span class="token string">'Permission'</span><span class="token punctuation">.</span> <span class="token variable">$permission</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">name</span><span class="token punctuation">.</span><span class="token string">' added!'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

        <span class="token punctuation">}</span>

        <span class="token comment">/**
        * Display the specified resource.
        *
        * @param  int  $id
        * @return \Illuminate\Http\Response
        */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">show</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token keyword">return</span> <span class="token function">redirect</span><span class="token punctuation">(</span><span class="token string">'permissions'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
        * Show the form for editing the specified resource.
        *
        * @param  int  $id
        * @return \Illuminate\Http\Response
        */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">edit</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$permission</span> <span class="token operator">=</span> Permission<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">findOrFail</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

            <span class="token keyword">return</span> <span class="token function">view</span><span class="token punctuation">(</span><span class="token string">'permissions.edit'</span><span class="token punctuation">,</span> <span class="token function">compact</span><span class="token punctuation">(</span><span class="token string">'permission'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
        * Update the specified resource in storage.
        *
        * @param  \Illuminate\Http\Request  $request
        * @param  int  $id
        * @return \Illuminate\Http\Response
        */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">update</span><span class="token punctuation">(</span>Request <span class="token variable">$request</span><span class="token punctuation">,</span> <span class="token variable">$id</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$permission</span> <span class="token operator">=</span> Permission<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">findOrFail</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
            <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">validate</span><span class="token punctuation">(</span><span class="token variable">$request</span><span class="token punctuation">,</span> <span class="token punctuation">[</span>
                <span class="token string">'name'</span><span class="token operator">=</span><span class="token operator">&gt;</span><span class="token string">'required|max:40'</span><span class="token punctuation">,</span>
            <span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
            <span class="token variable">$input</span> <span class="token operator">=</span> <span class="token variable">$request</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">all</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
            <span class="token variable">$permission</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">fill</span><span class="token punctuation">(</span><span class="token variable">$input</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">save</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

            <span class="token keyword">return</span> <span class="token function">redirect</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">route</span><span class="token punctuation">(</span><span class="token string">'permissions.index'</span><span class="token punctuation">)</span>
                <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">with</span><span class="token punctuation">(</span><span class="token string">'flash_message'</span><span class="token punctuation">,</span>
                 <span class="token string">'Permission'</span><span class="token punctuation">.</span> <span class="token variable">$permission</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">name</span><span class="token punctuation">.</span><span class="token string">' updated!'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

        <span class="token punctuation">}</span>

        <span class="token comment">/**
        * Remove the specified resource from storage.
        *
        * @param  int  $id
        * @return \Illuminate\Http\Response
        */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">destroy</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$permission</span> <span class="token operator">=</span> Permission<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">findOrFail</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

        <span class="token comment">//Make it impossible to delete this specific permission </span>
        <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token variable">$permission</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">name</span> <span class="token operator">==</span> <span class="token string">"Administer roles &amp; permissions"</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
                <span class="token keyword">return</span> <span class="token function">redirect</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">route</span><span class="token punctuation">(</span><span class="token string">'permissions.index'</span><span class="token punctuation">)</span>
                <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">with</span><span class="token punctuation">(</span><span class="token string">'flash_message'</span><span class="token punctuation">,</span>
                 <span class="token string">'Cannot delete this Permission!'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
            <span class="token punctuation">}</span>

            <span class="token variable">$permission</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">delete</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

            <span class="token keyword">return</span> <span class="token function">redirect</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">route</span><span class="token punctuation">(</span><span class="token string">'permissions.index'</span><span class="token punctuation">)</span>
                <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">with</span><span class="token punctuation">(</span><span class="token string">'flash_message'</span><span class="token punctuation">,</span>
                 <span class="token string">'Permission deleted!'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

        <span class="token punctuation">}</span>
    <span class="token punctuation">}</span></span>`</pre>

    In the `store()` method, we are making it possible for a `role` to be selected as a `permission` is created. After validating and saving the `permission` name field, a check is done if a `role` was selected if it was, a `permission` is assigned to the selected `role`.

    ### Permission View

    Three views are needed here as well. The `index` view would list in a table all the available permissions, the `create` view is a form which would be used to create a new `permission` and the edit view is a form that let's us edit existing `permission`.

    <pre data-title="html" class=" language-html">`{{-- \resources\views\permissions\index.blade.php --}}
    @extends('layouts.app')

    @section('title', '| Permissions')

    @section('content')

    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>col-lg-10 col-lg-offset-1<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>h1</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>i</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>fa fa-key<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>i</span><span class="token punctuation">&gt;</span></span>Available Permissions

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ route(<span class="token punctuation">'</span>users.index<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>btn btn-default pull-right<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>Users<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ route(<span class="token punctuation">'</span>roles.index<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>btn btn-default pull-right<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>Roles<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>h1</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>hr</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>table-responsive<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>table</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>table table-bordered table-striped<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>

                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>thead</span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>tr</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>th</span><span class="token punctuation">&gt;</span></span>Permissions<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>th</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>th</span><span class="token punctuation">&gt;</span></span>Operation<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>th</span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>tr</span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>thead</span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>tbody</span><span class="token punctuation">&gt;</span></span>
                    @foreach ($permissions as $permission)
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>tr</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>td</span><span class="token punctuation">&gt;</span></span>{{ $permission-&gt;name }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>td</span><span class="token punctuation">&gt;</span></span> 
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>td</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ URL::to(<span class="token punctuation">'</span>permissions/<span class="token punctuation">'</span>.$permission-&gt;id.<span class="token punctuation">'</span>/edit<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>btn btn-info pull-left<span class="token punctuation">"</span></span> <span class="token attr-name">style</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>margin-right: 3px;<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>Edit<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span>

                        {!! Form::open(['method' =&gt; 'DELETE', 'route' =&gt; ['permissions.destroy', $permission-&gt;id] ]) !!}
                        {!! Form::submit('Delete', ['class' =&gt; 'btn btn-danger']) !!}
                        {!! Form::close() !!}

                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>td</span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>tr</span><span class="token punctuation">&gt;</span></span>
                    @endforeach
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>tbody</span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>table</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ URL::to(<span class="token punctuation">'</span>permissions/create<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>btn btn-success<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>Add Permission<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span>

    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

    @endsection`</pre>

    The following is the `create` view

    <pre data-title="html" class=" language-html">`{{-- \resources\views\permissions\create.blade.php --}}
    @extends('layouts.app')

    @section('title', '| Create Permission')

    @section('content')

    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">'</span>col-lg-4 col-lg-offset-4<span class="token punctuation">'</span></span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>h1</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>i</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">'</span>fa fa-key<span class="token punctuation">'</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>i</span><span class="token punctuation">&gt;</span></span> Add Permission<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>h1</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>

        {{ Form::open(array('url' =&gt; 'permissions')) }}

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>form-group<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            {{ Form::label('name', 'Name') }}
            {{ Form::text('name', '', array('class' =&gt; 'form-control')) }}
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>
        @if(!$roles-&gt;isEmpty()) //If no roles exist yet
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>h4</span><span class="token punctuation">&gt;</span></span>Assign Permission to Roles<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>h4</span><span class="token punctuation">&gt;</span></span>

            @foreach ($roles as $role) 
                {{ Form::checkbox('roles[]',  $role-&gt;id ) }}
                {{ Form::label($role-&gt;name, ucfirst($role-&gt;name)) }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>

            @endforeach
        @endif
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>
        {{ Form::submit('Add', array('class' =&gt; 'btn btn-primary')) }}

        {{ Form::close() }}

    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

    @endsection
    `</pre>

    And finally the `edit` view:

    <pre data-title="html" class=" language-html">`@extends('layouts.app')

    @section('title', '| Edit Permission')

    @section('content')

    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">'</span>col-lg-4 col-lg-offset-4<span class="token punctuation">'</span></span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>h1</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>i</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">'</span>fa fa-key<span class="token punctuation">'</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>i</span><span class="token punctuation">&gt;</span></span> Edit {{$permission-&gt;name}}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>h1</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>
        {{ Form::model($permission, array('route' =&gt; array('permissions.update', $permission-&gt;id), 'method' =&gt; 'PUT')) }}{{-- Form model binding to automatically populate our fields with permission data --}}

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>form-group<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            {{ Form::label('name', 'Permission Name') }}
            {{ Form::text('name', null, array('class' =&gt; 'form-control')) }}
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>
        {{ Form::submit('Edit', array('class' =&gt; 'btn btn-primary')) }}

        {{ Form::close() }}

    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

    @endsection`</pre>

    ### Role Controller

    The `RoleController` is quite similar to the `UserController`. This controller will allow us to create `roles` and assign one or more `permissions` to a `role`. Create the file and paste the following code:

    <pre data-title="php" class=" language-php">`<span class="token php language-php"><span class="token delimiter important">&lt;?php</span>

    <span class="token keyword">namespace</span> <span class="token package">App<span class="token punctuation">\</span>Http<span class="token punctuation">\</span>Controllers</span><span class="token punctuation">;</span>

    <span class="token keyword">use</span> <span class="token package">Illuminate<span class="token punctuation">\</span>Http<span class="token punctuation">\</span>Request</span><span class="token punctuation">;</span>

    <span class="token keyword">use</span> <span class="token package">Auth</span><span class="token punctuation">;</span>
    <span class="token comment">//Importing laravel-permission models</span>
    <span class="token keyword">use</span> <span class="token package">Spatie<span class="token punctuation">\</span>Permission<span class="token punctuation">\</span>Models<span class="token punctuation">\</span>Role</span><span class="token punctuation">;</span>
    <span class="token keyword">use</span> <span class="token package">Spatie<span class="token punctuation">\</span>Permission<span class="token punctuation">\</span>Models<span class="token punctuation">\</span>Permission</span><span class="token punctuation">;</span>

    <span class="token keyword">use</span> <span class="token package">Session</span><span class="token punctuation">;</span>

    <span class="token keyword">class</span> <span class="token class-name">RoleController</span> <span class="token keyword">extends</span> <span class="token class-name">Controller</span> <span class="token punctuation">{</span>

        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">__construct</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">middleware</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token string">'auth'</span><span class="token punctuation">,</span> <span class="token string">'isAdmin'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span><span class="token comment">//isAdmin middleware lets only users with a //specific permission permission to access these resources</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
         * Display a listing of the resource.
         *
         * @return \Illuminate\Http\Response
         */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">index</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$roles</span> <span class="token operator">=</span> Role<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">all</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span><span class="token comment">//Get all roles</span>

            <span class="token keyword">return</span> <span class="token function">view</span><span class="token punctuation">(</span><span class="token string">'roles.index'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">with</span><span class="token punctuation">(</span><span class="token string">'roles'</span><span class="token punctuation">,</span> <span class="token variable">$roles</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
         * Show the form for creating a new resource.
         *
         * @return \Illuminate\Http\Response
         */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$permissions</span> <span class="token operator">=</span> Permission<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">all</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span><span class="token comment">//Get all permissions</span>

            <span class="token keyword">return</span> <span class="token function">view</span><span class="token punctuation">(</span><span class="token string">'roles.create'</span><span class="token punctuation">,</span> <span class="token punctuation">[</span><span class="token string">'permissions'</span><span class="token operator">=</span><span class="token operator">&gt;</span><span class="token variable">$permissions</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
         * Store a newly created resource in storage.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">store</span><span class="token punctuation">(</span>Request <span class="token variable">$request</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token comment">//Validate name and permissions field</span>
            <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">validate</span><span class="token punctuation">(</span><span class="token variable">$request</span><span class="token punctuation">,</span> <span class="token punctuation">[</span>
                <span class="token string">'name'</span><span class="token operator">=</span><span class="token operator">&gt;</span><span class="token string">'required|unique:roles|max:10'</span><span class="token punctuation">,</span>
                <span class="token string">'permissions'</span> <span class="token operator">=</span><span class="token operator">&gt;</span><span class="token string">'required'</span><span class="token punctuation">,</span>
                <span class="token punctuation">]</span>
            <span class="token punctuation">)</span><span class="token punctuation">;</span>

            <span class="token variable">$name</span> <span class="token operator">=</span> <span class="token variable">$request</span><span class="token punctuation">[</span><span class="token string">'name'</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
            <span class="token variable">$role</span> <span class="token operator">=</span> <span class="token keyword">new</span> <span class="token class-name">Role</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
            <span class="token variable">$role</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">name</span> <span class="token operator">=</span> <span class="token variable">$name</span><span class="token punctuation">;</span>

            <span class="token variable">$permissions</span> <span class="token operator">=</span> <span class="token variable">$request</span><span class="token punctuation">[</span><span class="token string">'permissions'</span><span class="token punctuation">]</span><span class="token punctuation">;</span>

            <span class="token variable">$role</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">save</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token comment">//Looping thru selected permissions</span>
            <span class="token keyword">foreach</span> <span class="token punctuation">(</span><span class="token variable">$permissions</span> <span class="token keyword">as</span> <span class="token variable">$permission</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
                <span class="token variable">$p</span> <span class="token operator">=</span> Permission<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">where</span><span class="token punctuation">(</span><span class="token string">'id'</span><span class="token punctuation">,</span> <span class="token string">'='</span><span class="token punctuation">,</span> <span class="token variable">$permission</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">firstOrFail</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> 
             <span class="token comment">//Fetch the newly created role and assign permission</span>
                <span class="token variable">$role</span> <span class="token operator">=</span> Role<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">where</span><span class="token punctuation">(</span><span class="token string">'name'</span><span class="token punctuation">,</span> <span class="token string">'='</span><span class="token punctuation">,</span> <span class="token variable">$name</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">first</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> 
                <span class="token variable">$role</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">givePermissionTo</span><span class="token punctuation">(</span><span class="token variable">$p</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
            <span class="token punctuation">}</span>

            <span class="token keyword">return</span> <span class="token function">redirect</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">route</span><span class="token punctuation">(</span><span class="token string">'roles.index'</span><span class="token punctuation">)</span>
                <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">with</span><span class="token punctuation">(</span><span class="token string">'flash_message'</span><span class="token punctuation">,</span>
                 <span class="token string">'Role'</span><span class="token punctuation">.</span> <span class="token variable">$role</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">name</span><span class="token punctuation">.</span><span class="token string">' added!'</span><span class="token punctuation">)</span><span class="token punctuation">;</span> 
        <span class="token punctuation">}</span>

        <span class="token comment">/**
         * Display the specified resource.
         *
         * @param  int  $id
         * @return \Illuminate\Http\Response
         */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">show</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token keyword">return</span> <span class="token function">redirect</span><span class="token punctuation">(</span><span class="token string">'roles'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
         * Show the form for editing the specified resource.
         *
         * @param  int  $id
         * @return \Illuminate\Http\Response
         */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">edit</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
            <span class="token variable">$role</span> <span class="token operator">=</span> Role<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">findOrFail</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
            <span class="token variable">$permissions</span> <span class="token operator">=</span> Permission<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">all</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

            <span class="token keyword">return</span> <span class="token function">view</span><span class="token punctuation">(</span><span class="token string">'roles.edit'</span><span class="token punctuation">,</span> <span class="token function">compact</span><span class="token punctuation">(</span><span class="token string">'role'</span><span class="token punctuation">,</span> <span class="token string">'permissions'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
         * Update the specified resource in storage.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  int  $id
         * @return \Illuminate\Http\Response
         */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">update</span><span class="token punctuation">(</span>Request <span class="token variable">$request</span><span class="token punctuation">,</span> <span class="token variable">$id</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>

            <span class="token variable">$role</span> <span class="token operator">=</span> Role<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">findOrFail</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span><span class="token punctuation">;</span><span class="token comment">//Get role with the given id</span>
        <span class="token comment">//Validate name and permission fields</span>
            <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">validate</span><span class="token punctuation">(</span><span class="token variable">$request</span><span class="token punctuation">,</span> <span class="token punctuation">[</span>
                <span class="token string">'name'</span><span class="token operator">=</span><span class="token operator">&gt;</span><span class="token string">'required|max:10|unique:roles,name,'</span><span class="token punctuation">.</span><span class="token variable">$id</span><span class="token punctuation">,</span>
                <span class="token string">'permissions'</span> <span class="token operator">=</span><span class="token operator">&gt;</span><span class="token string">'required'</span><span class="token punctuation">,</span>
            <span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

            <span class="token variable">$input</span> <span class="token operator">=</span> <span class="token variable">$request</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">except</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token string">'permissions'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
            <span class="token variable">$permissions</span> <span class="token operator">=</span> <span class="token variable">$request</span><span class="token punctuation">[</span><span class="token string">'permissions'</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
            <span class="token variable">$role</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">fill</span><span class="token punctuation">(</span><span class="token variable">$input</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">save</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

            <span class="token variable">$p_all</span> <span class="token operator">=</span> Permission<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">all</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span><span class="token comment">//Get all permissions</span>

            <span class="token keyword">foreach</span> <span class="token punctuation">(</span><span class="token variable">$p_all</span> <span class="token keyword">as</span> <span class="token variable">$p</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
                <span class="token variable">$role</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">revokePermissionTo</span><span class="token punctuation">(</span><span class="token variable">$p</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//Remove all permissions associated with role</span>
            <span class="token punctuation">}</span>

            <span class="token keyword">foreach</span> <span class="token punctuation">(</span><span class="token variable">$permissions</span> <span class="token keyword">as</span> <span class="token variable">$permission</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
                <span class="token variable">$p</span> <span class="token operator">=</span> Permission<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">where</span><span class="token punctuation">(</span><span class="token string">'id'</span><span class="token punctuation">,</span> <span class="token string">'='</span><span class="token punctuation">,</span> <span class="token variable">$permission</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">firstOrFail</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//Get corresponding form //permission in db</span>
                <span class="token variable">$role</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">givePermissionTo</span><span class="token punctuation">(</span><span class="token variable">$p</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  <span class="token comment">//Assign permission to role</span>
            <span class="token punctuation">}</span>

            <span class="token keyword">return</span> <span class="token function">redirect</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">route</span><span class="token punctuation">(</span><span class="token string">'roles.index'</span><span class="token punctuation">)</span>
                <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">with</span><span class="token punctuation">(</span><span class="token string">'flash_message'</span><span class="token punctuation">,</span>
                 <span class="token string">'Role'</span><span class="token punctuation">.</span> <span class="token variable">$role</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">name</span><span class="token punctuation">.</span><span class="token string">' updated!'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/**
         * Remove the specified resource from storage.
         *
         * @param  int  $id
         * @return \Illuminate\Http\Response
         */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">destroy</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span>
        <span class="token punctuation">{</span>
            <span class="token variable">$role</span> <span class="token operator">=</span> Role<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">findOrFail</span><span class="token punctuation">(</span><span class="token variable">$id</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
            <span class="token variable">$role</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">delete</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

            <span class="token keyword">return</span> <span class="token function">redirect</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">route</span><span class="token punctuation">(</span><span class="token string">'roles.index'</span><span class="token punctuation">)</span>
                <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">with</span><span class="token punctuation">(</span><span class="token string">'flash_message'</span><span class="token punctuation">,</span>
                 <span class="token string">'Role deleted!'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

        <span class="token punctuation">}</span>
    <span class="token punctuation">}</span></span>`</pre>

    ### Roles View

    Three views are needed here as well. The `index` view to display available `roles` and associated `permissions`, the `create` view to add a new `role` and a view to edit an existing `role`. Create the `index.blade.php` file and paste the following:

    <pre data-title="html" class=" language-html">`{{-- \resources\views\roles\index.blade.php --}}
    @extends('layouts.app')

    @section('title', '| Roles')

    @section('content')

    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>col-lg-10 col-lg-offset-1<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>h1</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>i</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>fa fa-key<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>i</span><span class="token punctuation">&gt;</span></span> Roles

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ route(<span class="token punctuation">'</span>users.index<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>btn btn-default pull-right<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>Users<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ route(<span class="token punctuation">'</span>permissions.index<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>btn btn-default pull-right<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>Permissions<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>h1</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>hr</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>table-responsive<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>table</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>table table-bordered table-striped<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>thead</span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>tr</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>th</span><span class="token punctuation">&gt;</span></span>Role<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>th</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>th</span><span class="token punctuation">&gt;</span></span>Permissions<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>th</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>th</span><span class="token punctuation">&gt;</span></span>Operation<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>th</span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>tr</span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>thead</span><span class="token punctuation">&gt;</span></span>

                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>tbody</span><span class="token punctuation">&gt;</span></span>
                    @foreach ($roles as $role)
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>tr</span><span class="token punctuation">&gt;</span></span>

                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>td</span><span class="token punctuation">&gt;</span></span>{{ $role-&gt;name }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>td</span><span class="token punctuation">&gt;</span></span>

                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>td</span><span class="token punctuation">&gt;</span></span>{{ str_replace(array('[',']','"'),'', $role-&gt;permissions()-&gt;pluck('name')) }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>td</span><span class="token punctuation">&gt;</span></span>{{-- Retrieve array of permissions associated to a role and convert to string --}}
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>td</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ URL::to(<span class="token punctuation">'</span>roles/<span class="token punctuation">'</span>.$role-&gt;id.<span class="token punctuation">'</span>/edit<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>btn btn-info pull-left<span class="token punctuation">"</span></span> <span class="token attr-name">style</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>margin-right: 3px;<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>Edit<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span>

                        {!! Form::open(['method' =&gt; 'DELETE', 'route' =&gt; ['roles.destroy', $role-&gt;id] ]) !!}
                        {!! Form::submit('Delete', ['class' =&gt; 'btn btn-danger']) !!}
                        {!! Form::close() !!}

                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>td</span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>tr</span><span class="token punctuation">&gt;</span></span>
                    @endforeach
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>tbody</span><span class="token punctuation">&gt;</span></span>

            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>table</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{{ URL::to(<span class="token punctuation">'</span>roles/create<span class="token punctuation">'</span>) }}<span class="token punctuation">"</span></span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>btn btn-success<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>Add Role<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span>

    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

    @endsection`</pre>

    For the `create` view:

    <pre data-title="html" class=" language-html">`@extends('layouts.app')

    @section('title', '| Add Role')

    @section('content')

    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">'</span>col-lg-4 col-lg-offset-4<span class="token punctuation">'</span></span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>h1</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>i</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">'</span>fa fa-key<span class="token punctuation">'</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>i</span><span class="token punctuation">&gt;</span></span> Add Role<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>h1</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>hr</span><span class="token punctuation">&gt;</span></span>

        {{ Form::open(array('url' =&gt; 'roles')) }}

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>form-group<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            {{ Form::label('name', 'Name') }}
            {{ Form::text('name', null, array('class' =&gt; 'form-control')) }}
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>h5</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>b</span><span class="token punctuation">&gt;</span></span>Assign Permissions<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>b</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>h5</span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">'</span>form-group<span class="token punctuation">'</span></span><span class="token punctuation">&gt;</span></span>
            @foreach ($permissions as $permission)
                {{ Form::checkbox('permissions[]',  $permission-&gt;id ) }}
                {{ Form::label($permission-&gt;name, ucfirst($permission-&gt;name)) }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>

            @endforeach
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

        {{ Form::submit('Add', array('class' =&gt; 'btn btn-primary')) }}

        {{ Form::close() }}

    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

    @endsection`</pre>

    And for the `edit` view:

    <pre data-title="html" class=" language-html">`@extends('layouts.app')

    @section('title', '| Edit Role')

    @section('content')

    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">'</span>col-lg-4 col-lg-offset-4<span class="token punctuation">'</span></span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>h1</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>i</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">'</span>fa fa-key<span class="token punctuation">'</span></span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>i</span><span class="token punctuation">&gt;</span></span> Edit Role: {{$role-&gt;name}}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>h1</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>hr</span><span class="token punctuation">&gt;</span></span>

        {{ Form::model($role, array('route' =&gt; array('roles.update', $role-&gt;id), 'method' =&gt; 'PUT')) }}

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>form-group<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            {{ Form::label('name', 'Role Name') }}
            {{ Form::text('name', null, array('class' =&gt; 'form-control')) }}
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>h5</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>b</span><span class="token punctuation">&gt;</span></span>Assign Permissions<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>b</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>h5</span><span class="token punctuation">&gt;</span></span>
        @foreach ($permissions as $permission)

            {{Form::checkbox('permissions[]',  $permission-&gt;id, $role-&gt;permissions ) }}
            {{Form::label($permission-&gt;name, ucfirst($permission-&gt;name)) }}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>

        @endforeach
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>
        {{ Form::submit('Edit', array('class' =&gt; 'btn btn-primary')) }}

        {{ Form::close() }}    
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

    @endsection`</pre>

    ### Middleware

    To restrict access to the `roles` and `permissions` page, a middleware was included called `isAdmin` in our `PermissionController` and `RoleController`. This middleware counts how many users are in the Users table, and if there are more than one users, it checks if the current authenticated User has the permission to 'Administer roles &amp; permissions'. To create a permission visit `http://localhost:8000/permissions/create`. Then go to `http://localhost:8000/roles/create` to create a `role`, to which you can now assign the `permission` you created. For example you can create a permission called 'Administer roles &amp; permissions' and a 'Admin' `role` to which you would assign this `permission`. Create the `AdminMiddleware` in the directory `app/Http/Middleware/` and enter the following code:

    <pre data-title="php" class=" language-php">`<span class="token php language-php"><span class="token delimiter important">&lt;?php</span>

    <span class="token keyword">namespace</span> <span class="token package">App<span class="token punctuation">\</span>Http<span class="token punctuation">\</span>Middleware</span><span class="token punctuation">;</span>

    <span class="token keyword">use</span> <span class="token package">Closure</span><span class="token punctuation">;</span>
    <span class="token keyword">use</span> <span class="token package">Illuminate<span class="token punctuation">\</span>Support<span class="token punctuation">\</span>Facades<span class="token punctuation">\</span>Auth</span><span class="token punctuation">;</span>
    <span class="token keyword">use</span> <span class="token package">App<span class="token punctuation">\</span>User</span><span class="token punctuation">;</span>

    <span class="token keyword">class</span> <span class="token class-name">AdminMiddleware</span>
    <span class="token punctuation">{</span>
        <span class="token comment">/**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">handle</span><span class="token punctuation">(</span><span class="token variable">$request</span><span class="token punctuation">,</span> Closure <span class="token variable">$next</span><span class="token punctuation">)</span>
        <span class="token punctuation">{</span>
            <span class="token variable">$user</span> <span class="token operator">=</span> User<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">all</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">count</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
            <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span><span class="token punctuation">(</span><span class="token variable">$user</span> <span class="token operator">==</span> <span class="token number">1</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
                <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span>Auth<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">user</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">hasPermissionTo</span><span class="token punctuation">(</span><span class="token string">'Administer roles &amp; permissions'</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token comment">//If user does //not have this permission</span>
            <span class="token punctuation">{</span>
                    <span class="token function">abort</span><span class="token punctuation">(</span><span class="token string">'401'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
                <span class="token punctuation">}</span>
            <span class="token punctuation">}</span>

            <span class="token keyword">return</span> <span class="token variable">$next</span><span class="token punctuation">(</span><span class="token variable">$request</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>
    <span class="token punctuation">}</span></span>`</pre>

    A middleware called `clearance` was also included in our `PostController`. This middleware would check if a `user` has the `permissions` Administer roles &amp; permissions, Create Post, Edit Post and Delete Post.

    <pre data-title="php" class=" language-php">`<span class="token php language-php"><span class="token delimiter important">&lt;?php</span>

    <span class="token keyword">namespace</span> <span class="token package">App<span class="token punctuation">\</span>Http<span class="token punctuation">\</span>Middleware</span><span class="token punctuation">;</span>

    <span class="token keyword">use</span> <span class="token package">Closure</span><span class="token punctuation">;</span>
    <span class="token keyword">use</span> <span class="token package">Illuminate<span class="token punctuation">\</span>Support<span class="token punctuation">\</span>Facades<span class="token punctuation">\</span>Auth</span><span class="token punctuation">;</span>

    <span class="token keyword">class</span> <span class="token class-name">ClearanceMiddleware</span> <span class="token punctuation">{</span>
        <span class="token comment">/**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */</span>
        <span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">handle</span><span class="token punctuation">(</span><span class="token variable">$request</span><span class="token punctuation">,</span> Closure <span class="token variable">$next</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>        
            <span class="token keyword">if</span> <span class="token punctuation">(</span>Auth<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">user</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">hasPermissionTo</span><span class="token punctuation">(</span><span class="token string">'Administer roles &amp; permissions'</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token comment">//If user has this //permission</span>
        <span class="token punctuation">{</span>
                <span class="token keyword">return</span> <span class="token variable">$next</span><span class="token punctuation">(</span><span class="token variable">$request</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
            <span class="token punctuation">}</span>

            <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token variable">$request</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">is</span><span class="token punctuation">(</span><span class="token string">'posts/create'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token comment">//If user is creating a post</span>
             <span class="token punctuation">{</span>
                <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span>Auth<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">user</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">hasPermissionTo</span><span class="token punctuation">(</span><span class="token string">'Create Post'</span><span class="token punctuation">)</span><span class="token punctuation">)</span>
             <span class="token punctuation">{</span>
                    <span class="token function">abort</span><span class="token punctuation">(</span><span class="token string">'401'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
                <span class="token punctuation">}</span> 
             <span class="token keyword">else</span> <span class="token punctuation">{</span>
                    <span class="token keyword">return</span> <span class="token variable">$next</span><span class="token punctuation">(</span><span class="token variable">$request</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
                <span class="token punctuation">}</span>
            <span class="token punctuation">}</span>

            <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token variable">$request</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">is</span><span class="token punctuation">(</span><span class="token string">'posts/*/edit'</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token comment">//If user is editing a post</span>
             <span class="token punctuation">{</span>
                <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span>Auth<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">user</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">hasPermissionTo</span><span class="token punctuation">(</span><span class="token string">'Edit Post'</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
                    <span class="token function">abort</span><span class="token punctuation">(</span><span class="token string">'401'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
                <span class="token punctuation">}</span> <span class="token keyword">else</span> <span class="token punctuation">{</span>
                    <span class="token keyword">return</span> <span class="token variable">$next</span><span class="token punctuation">(</span><span class="token variable">$request</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
                <span class="token punctuation">}</span>
            <span class="token punctuation">}</span>

            <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token variable">$request</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">isMethod</span><span class="token punctuation">(</span><span class="token string">'Delete'</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token comment">//If user is deleting a post</span>
             <span class="token punctuation">{</span>
                <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span>Auth<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">user</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">hasPermissionTo</span><span class="token punctuation">(</span><span class="token string">'Delete Post'</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
                    <span class="token function">abort</span><span class="token punctuation">(</span><span class="token string">'401'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
                <span class="token punctuation">}</span> 
             <span class="token keyword">else</span> 
             <span class="token punctuation">{</span>
                    <span class="token keyword">return</span> <span class="token variable">$next</span><span class="token punctuation">(</span><span class="token variable">$request</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
                <span class="token punctuation">}</span>
            <span class="token punctuation">}</span>

            <span class="token keyword">return</span> <span class="token variable">$next</span><span class="token punctuation">(</span><span class="token variable">$request</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>
    <span class="token punctuation">}</span></span>`</pre>

    Add `AdminMiddleware::class` and `ClearanceMiddleware::class` to the `$routeMiddleware` property of `/app/Http/kernel.php` like this:

    <pre data-title="php" class=" language-php">`<span class="token keyword">protected</span> <span class="token variable">$routeMiddleware</span> <span class="token operator">=</span> <span class="token punctuation">[</span>
            <span class="token string">'auth'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> \<span class="token package">Illuminate<span class="token punctuation">\</span>Auth<span class="token punctuation">\</span>Middleware<span class="token punctuation">\</span>Authenticate</span><span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token keyword">class</span><span class="token punctuation">,</span>
            <span class="token string">'auth.basic'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> \<span class="token package">Illuminate<span class="token punctuation">\</span>Auth<span class="token punctuation">\</span>Middleware<span class="token punctuation">\</span>AuthenticateWithBasicAuth</span><span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token keyword">class</span><span class="token punctuation">,</span>
            <span class="token string">'bindings'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> \<span class="token package">Illuminate<span class="token punctuation">\</span>Routing<span class="token punctuation">\</span>Middleware<span class="token punctuation">\</span>SubstituteBindings</span><span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token keyword">class</span><span class="token punctuation">,</span>
            <span class="token string">'can'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> \<span class="token package">Illuminate<span class="token punctuation">\</span>Auth<span class="token punctuation">\</span>Middleware<span class="token punctuation">\</span>Authorize</span><span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token keyword">class</span><span class="token punctuation">,</span>
            <span class="token string">'guest'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> \<span class="token package">App<span class="token punctuation">\</span>Http<span class="token punctuation">\</span>Middleware<span class="token punctuation">\</span>RedirectIfAuthenticated</span><span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token keyword">class</span><span class="token punctuation">,</span>
            <span class="token string">'throttle'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> \<span class="token package">Illuminate<span class="token punctuation">\</span>Routing<span class="token punctuation">\</span>Middleware<span class="token punctuation">\</span>ThrottleRequests</span><span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token keyword">class</span><span class="token punctuation">,</span>
            <span class="token string">'isAdmin'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> \<span class="token package">App<span class="token punctuation">\</span>Http<span class="token punctuation">\</span>Middleware<span class="token punctuation">\</span>AdminMiddleware</span><span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token keyword">class</span><span class="token punctuation">,</span>
            <span class="token string">'clearance'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> \<span class="token package">App<span class="token punctuation">\</span>Http<span class="token punctuation">\</span>Middleware<span class="token punctuation">\</span>ClearanceMiddleware</span><span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token keyword">class</span><span class="token punctuation">,</span>
    <span class="token punctuation">]</span><span class="token punctuation">;</span>`</pre>

    In both middelwares a 401 exception would be thrown if the conditions are not meet. Let's create a custom 401 error page:

    <pre data-title="html" class=" language-html">`{{-- \resources\views\errors\401.blade.php --}}
    @extends('layouts.app')

    @section('content')
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">'</span>col-lg-4 col-lg-offset-4<span class="token punctuation">'</span></span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>h1</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>center</span><span class="token punctuation">&gt;</span></span>401<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>br</span><span class="token punctuation">&gt;</span></span>
            ACCESS DENIED<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>center</span><span class="token punctuation">&gt;</span></span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>h1</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>

    @endsection

### Wrapping Up

First lets create an 'Admin' user and then create the necessary permissions and roles. Click on Register and create a user, then go to `http://localhost:8000/permissions` and create permissions to `Create Post`, `Edit Post`, `Delete Post` and `Administer roles &amp; permissions`. After creating these permissions, your permissions page should look like this:

![](https://cdn.scotch.io/21141/Mnh3hoKhSJyOvwXII25b_permissions.png)

Next, you need to create `roles` to which you would add the Create, Edit and Delete Permissions. Click on Roles and create these `roles`:

*   Admin- A user assigned to this role would have all permissions
*   Owner- A user assigned to this role would have selected permissions assigned to it by Admin

![](https://cdn.scotch.io/21141/rlECwvxQOOV5GLty2EJu_Roles.png)

Finally assign the Role of 'Admin' to the currently logged in User. Click on Users and then Edit. Check the Admin box under Give Role:

![](https://cdn.scotch.io/21141/BojnrM1ASS7QfBxWcVHQ_edit.png)

After assigning the 'Admin' `role` to our `user`, notice that you now have a new Admin link in the drop of the navigation, this links to our users page. Now create a new `user` and give it the more restrictive `role` of Owner. If you login as this user and try to visit the User, Role or Permission pages you get this as expected:

![](https://cdn.scotch.io/21141/aNE94YaHScqQgzx9Lr3J_denied.png)

The Owner `role` does not have permission to `Administer Roles &amp; Users` hence the exception is thrown. 

To demonstrate how this works for `posts`, create a `post` by clicking on New Article. After creating the post, view the `post` and you would notice you have along with the Back button, an Edit and Delete button as shown below:

![](https://cdn.scotch.io/21141/pZh6YbquQOOxgYZuCdmR_post.png)

Now if you logout and view the post only the Back button will be available to us. This also works if you have a logged in user who does not have permissions to Edit or Delete Post.

![](https://cdn.scotch.io/21141/maxduyJNRTaV1NAWxLsg_post2.png)

### Conclusion

The laravel-permission package makes it relatively easy to build a role and permission system. To recap we have considered installation of the laravel-permission package, laravel-permission blade directives, creating a custom middleware and implementing an access control list in a Laravel application. You can look at the final product on [Github](https://github.com/caleboki/acl) and if you have any questions or comments, don’t hesitate to post them below.
