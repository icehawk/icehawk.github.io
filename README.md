# IceHawk website

## Edit the website

The website is (obviously) generated static content.
 
If you want to make changes, fix typos or add missing documentation, please follow these steps.

### 1. Fork & clone the respository

* Please fork the repository to your github account.
* Clone the repository to you local machine

```bash
$ git clone https://github.com/<your-github-user>/icehawk.github.io.git
```

### 2. Install the static page generator

The static page generator is a PHAR that is installed using [composer](https://getcomposer.org) 
and [tm/tooly-composer-script](https://github.com/tommy-muehle/tooly-composer-script).

Simply run:

```bash
$ cd icehawk.github.io
$ mkdir -p vendor/bin
$ composer update
```

The static page generator PHAR is placed to `vendor/bin/spg.phar` and is executable.

### 3. Make it a Document Root of a webserver

The simplest way is to use the build-in PHP webserver, like this:

```bash
$ cd icehawk.github.io
$ php -S 127.0.0.1:8088
# Should print something like this
PHP 7.0.11 Development Server started at Fri Nov  4 11:03:32 2016
Listening on http://127.0.0.1:8088
Document root is /Users/hollodotme/Sites/icehawk.github.io
Press Ctrl-C to quit.
```

### 4. Generate the pages for your locale base URL

In order to generate the pages for your local URL (http://127.0.0.1:8088 or whatever your webserver's URL is), you need 
to run the static page generator with the option `--baseUrl="http://127.0.0.1:8088"`.

```bash
$ vendor/bin/spg.phar generate:pages --baseUrl="http://127.0.0.1:8088" ./Project.json
```

### 5. Open in browser

Now you should be able to view the pages in your browser when visiting http://127.0.0.1:8088.

### 6. Make changes

* You can edit the content files located in the `./Contents` folder.
* Please **do not edit** the `.html` files, those changes will be gone when generating the pages.
* You can add new pages in the `./Project.json` file. Please refer to the already existing page configs there.

### 7. Commit & push your changes

**THIS IS IMPORTANT!**

**Before you commit your changes**, please generate the pages again **without** the `--baseUrl`-option, 
so that the real base URL from the settings will take effect.

```bash
$ vendor/bin/spg.phar generate:pages ./Project.json
$ git add -A
$ git commit -m '...'
$ git push
```

### 8. Create a pull request

Please create a pull request to the origin repository. We'll then check your changes and merge them.

---

Thanks for your help on improving our docs, we really appreciate that!
