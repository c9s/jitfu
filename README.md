JIT
===
*An OO wrapper around libjit ...*

```libjit``` is a library for creating Just-In-Time compilers, this extension wraps the library such that it's functionality can be called from PHP.

This extension can be used to build native instructions in PHP ... and is fun to play with ...

```php
<?php
/* This is a fibonacci function, and is ~60 times faster than PHP :o */
use JIT\Context;
use JIT\Type;
use JIT\Signature;
use JIT\Func;
use JIT\Value;
use JIT\Builder;

$context = new Context();

$context->start();

$integer   = new Type(JIT_TYPE_LONG);
$signature = new Signature
	($integer, [$integer]);

$function = new Func($context, $signature);
$zero     = new Value($function, 0, $integer);
$one      = new Value($function, 1, $integer);
$two      = new Value($function, 2, $integer);
$three    = new Value($function, 3, $integer);

$arg      = $function->getParameter(0);

$builder  = new Builder($function);

/* if ($arg == 0) return 0; */
$builder->doIf(
	$builder->doEq($arg, $zero),
	function($builder) use ($arg, $zero, $one, $two) {
		$builder->doReturn($zero);
	}
);

/* if ($arg == 1) return 1; */
$builder->doIf(
	$builder->doEq($arg, $one),
	function($builder) use($one) {
		$builder->doReturn($one);
	}
);

/* return $function($arg-1) + $function($arg-2); */
$builder->doReturn(
	$builder->doAdd(
		$builder->doCall($function, [$builder->doSub($arg, $one)], 0),
		$builder->doCall($function, [$builder->doSub($arg, $two)], 0)));

$context->finish();

$function->compile();

/* now the function is compiled, it can be passed around like a callable ... */

var_dump(
	$context, 
	$signature,
	$function,
	$function(40)); /* __invoke with magicalness */
?>
```

**This library is not useful for compiling Zend opcodes**
