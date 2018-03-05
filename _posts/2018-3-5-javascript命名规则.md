The following are the rules for naming JavaScript variables:

1.
A variable name cannot start with a numeral. For instance, 3x or 2goats or 76trombones would all be illegal variable names.

You can, however, have numbers within a JavaScript variable name; for instance up2me or go4it would both be perfectly valid variable names.

2.
You cannot have a mathematical or logical operator in a variable name. For instance, 2*something or this+that would both be illegal... because the * and the + are arithmetic operators. The same holds true for ^, /, \, !, etc.

3.
You must not use any punctuation marks of any kind in a JavaScript variable name, other than the underscore; for example... some:thing or big# or do'to would all be illegal.

The underscore is the exception, and it can be used at the beginning, within, or at the end of JavaScript variable names. You can use names like _pounds or some_thing or gallons_ as variable names and they are perfectly legal.

4.
JavaScript names must not contain spaces. Ever.

5.
You cannot use JavaScript keywords (parts of the language, itself) for variable names. Thus window or open or location or string would be illegal.  Check a JavaScript reference if in doubt as to whether something is or is not part of the language -- JavaScript has grown into a fairly fully-fleshed language, so you may get some occasional surprises.

You can, of course, use what are otherwise keywords as parts of variable names. For instance, thatWindow or someString or theLocation would all be perfectly acceptable.

6.
JavaScript variable names are case-sensitive. Programmers in other languages are often tripped up by this one, as some languages are not sensitive to case in variable names.

For instance, all of the following names would be considered completely different variable names in JavaScript:  gasbag  Gasbag  GasBag  gasBag

7.
允许使用$或者_开头作为变量名，一般以下划线开头的变量表示为私有变量，以$开头的变量表示为全局变量