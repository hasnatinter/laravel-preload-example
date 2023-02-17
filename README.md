## Introduction

OP cache is a feature of PHP where the op codes are cached and directly executed into machine code. After the first execution of the file the code is saved into opcodes and if the file’s opcode is available then it is returned instead of parsing and make AST of PHP code for file.

<aside>
💡 PHP OP Cache is a product of zend. The previous cache framework was [APC](https://en.wikipedia.org/wiki/List_of_PHP_accelerators).

</aside>

Remember PHP code lifecycle is following

1. PHP code: The php script file, if an opcode exists then it is directly turned to machine code
2. Lexical/Token: Makes tokens from code
3. Parsing: From tokens make AST (abstract syntax trees)
4. Compilation: Return opcodes for code
5. Machine code: Zend engine like JVM converts opcode to machine code

<aside>
💡 OP cache is stored in RAM and by default has 128MB for space available
</aside>

## Op Cache preloading

Preloading is an extension of current opcache feature and is designed to be specifically used in production environment only.

### The Idea 💡

The idea is to to define a preload file where all the classes of a project(or a framework e.g. vendor folder) are to be pre-compiled in opcode and that code is served each request rather than

- Without opcache: Compile fie (parse and lex)
- With opcache: Check if the file was modified. Another overhead is e.g. a file is cached but its dependency is not, then it also has to be cached.

Preload will permanently load opcode of a PHP file in memory on server (PHP fpm) startup. From then on any modification will not be effected (since a cached version) is served.

### A sample PHP script to preload from PHP project like Zend
```php
<?php
function _preload($preload, string $pattern = "/\.php$/", array $ignore = []) {
	// if array of paths provided
  if (is_array($preload)) {
    foreach ($preload as $path) {
      _preload($path, $pattern, $ignore);
    }
  } else if (is_string($preload)) {
    $path = $preload;
		// if an array of paths (or files) to be ignored provided
    if (!in_array($path, $ignore)) {
			// if directory then 
      if (is_dir($path)) {
        if ($dh = opendir($path)) {
					// preload all files and paths in directory
          while (($file = readdir($dh)) !== false) {
            if ($file !== "." && $file !== "..") {
              _preload($path . "/" . $file, $pattern, $ignore);
            }
          }
          closedir($dh);
        }
			// if path and path is PHP ("/\.php$/") then add to preload cache
      } else if (is_file($path) && preg_match($pattern, $path)) {
        if (!opcache_compile_file($path)) {
          trigger_error("Preloading Failed", E_USER_ERROR);
        }
      }
    }
  }
}
```