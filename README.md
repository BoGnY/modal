<p align="center">
<a href="https://github.com/livewire-ui/modal/actions"><img src="https://github.com/livewire-ui/modal/workflows/PHPUnit/badge.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/livewire-ui/modal"><img src="https://img.shields.io/packagist/dt/livewire-ui/modal" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/livewire-ui/modal"><img src="https://img.shields.io/packagist/v/livewire-ui/modal" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/livewire-ui/modal"><img src="https://img.shields.io/packagist/l/livewire-ui/modal" alt="License"></a>
</p>

## About LivewireUI Modal
LivewireUI Modal is a Livewire component that provides you with a modal that supports multiple child modals while maintaining state.

## Installation
To get started, require the package via Composer:

```
composer require livewire-ui/modal
```

## Livewire directive
Add the Livewire directive `@livewire('livewire-ui-modal')` and also the Javascript `@livewireUIScripts` directive to your template.
```html
<html>
<body>
    <!-- content -->
    
    @livewire('livewire-ui-modal')
    @livewireUIScripts
</body>
</html>
```

Next you will need to publish the required scripts with the following command:
```shell
php artisan vendor:publish --tag=livewire-ui:public
```

## Alpine
Livewire UI requires [Alpine](https://github.com/alpinejs/alpine). You can use the official CDN to quickly include Alpine:

```html
<script src="https://cdn.jsdelivr.net/gh/alpinejs/alpine@v2.x.x/dist/alpine.min.js" defer></script>
```

## TailwindCSS
The base modal is made with TailwindCSS. If you use a different CSS framework I recommend that you publish the modal template and change the markup to include the required classes for your CSS framework.
```shell
php artisan vendor:publish --tag=livewire-ui:views
```


## Creating a modal
You can run `php artisan make:livewire EditUser` to make the initial Livewire component. Open your component class and make sure it extends the `ModalComponent` class:

```php
<?php

namespace App\Http\Livewire;

use LivewireUI\Modal\ModalComponent;

class EditUser extends ModalComponent
{
    public function render()
    {
        return view('livewire.edit-user');
    }
}
```

## Opening a modal
To open a modal you will need to emit an event. To open the `EditUser` modal for example:

```html
<!-- Outside of any Livewire component -->
<button onclick="Livewire.emit('openModal', 'edit-user')">Edit User</button>

<!-- Inside existing Livewire component -->
<button wire:click="$emit('openModal', 'edit-user')">Edit User</button>
```

## Passing parameters
To open the `EditUser` modal for a specific user we can pass the user id:

```html
<!-- Outside of any Livewire component -->
<button onclick="Livewire.emit('openModal', 'edit-user', @json(['user' => $user->id]))">Edit User</button>

<!-- Inside existing Livewire component -->
<button wire:click="$emit('openModal', 'edit-user', @json(['user' => $user->id])">Edit User</button>
```

The parameters are passed to the `mount` method on the modal component:

```php
<?php

namespace App\Http\Livewire;

use App\Models\User;
use LivewireUI\Modal\ModalComponent;

class EditUser extends ModalComponent
{
    public User $user;
    
    public function mount(User $user) {
        $this->user = $user;
    }
    
    public function render()
    {
        return view('livewire.edit-user');
    }
}
```

## Opening a child modal
From an existing modal you can use the exact same event and a child modal will be created:

```html
<!-- Edit User Modal -->

<!-- Edit Form -->

<button wire:click="$emit('openModal', 'delete-user', @json(['user' => $user->id])">Delete User</button>
```

## Closing a (child) modal
If for example a user clicks the 'Delete' button which will open a confirm dialog, you can cancel the deletion and return to the edit user modal by emitting the `closeModal` event. This will open the previous modal. If there is no previous modal the entire modal component is closed and the state will be reset.
```html
<button wire:click="$emit('closeModal')">No, do not delete</button>
```

You can also close a modal from within your modal component class:

```php
<?php

namespace App\Http\Livewire;

use App\Models\User;
use LivewireUI\Modal\ModalComponent;

class EditUser extends ModalComponent
{
    public User $user;

    public function mount(User $user) {
        $this->user = $user;
    }
    
    public function update()
    {
        $this->user->update($data);
        
        $this->closeModal();
    }

    public function render()
    {
        return view('livewire.edit-user');
    }
}
```

If you don't want to go to the previous modal but close the entire modal component you can use the `forceClose` method:

```php
public function update()
{
    $this->user->update($data);

    $this->forceClose()->closeModal();
}
```

Often you will want to update other Livewire components when changes have been made. For example, the user overview when a user is updated. You can use the `closeModalWithEvents` method to achieve this. 

```php
public function update()
{
    $this->user->update($data);

    $this->closeModalWithEvents([
        UserOverview::getName() => 'userModified',
    ]);
}
```

It's also possible to add parameters to your events:

```php
public function update()
{
    $this->user->update($data);

    $this->closeModalWithEvents([
        UserOverview::getName() => ['userModified', [$this->user->id],
    ]);
}
```

## Skipping previous modals
In some cases you might want to skip previous modals. For example:
1. Team overview modal
2. -> Edit Team
3. -> Delete Team

In this case, when a team is deleted, you don't want to go back to step 2 but go back to the overview.
You can use the `skipPreviousModal` method to achieve this. By default it will skip the previous modal. If you want to skip more you can pass the number of modals to skip `skipPreviousModals(2)`.

```php
<?php

namespace App\Http\Livewire;

use App\Models\Team;
use LivewireUI\Modal\ModalComponent;

class DeleteTeam extends ModalComponent
{
    public Team $team;

    public function mount(Team $team) {
        $this->team = $team;
    }

    public function delete()
    {
        $this->team->delete();

        $this->skipPreviousModal()->closeModalWithEvents([
            TeamOverview::getName() => 'teamDeleted'
        ]);
    }

    public function render()
    {
        return view('livewire.delete-team');
    }
}
```

## Credits
- [Philo Hermans](https://github.com/philoNL)
- [All Contributors](../../contributors)

## License
Livewire UI is open-sourced software licensed under the [MIT license](LICENSE.md).