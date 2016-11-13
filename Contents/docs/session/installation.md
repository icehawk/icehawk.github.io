# Session installation

<hr class="blockquote">

## Via icehawk/installer

```bash
$ php composer.phar create:project icehawk/installer /path/to/new/project
```

When you're asked if you want to install more IceHawk components, choose "yes" and then select "yes" for the session component. 

<hr class="blockquote">

## Via composer

```bash
$ php composer.phar require "icehawk/session:^1.0"
```

<hr class="blockquote">

## Manual in composer.json

```json
{
	"require": {
		"icehawk/session": "^1.0"
	}
}
```
