# CalcRS - Scientific calculator written in Rust

## Features:

- Focusing on simplicity and ease-of-use: Does what you would expect from a calculator.
- Written purely in Rust, using egui and complied with musl target to have a portable executable that works on any Linux. No dependencies or wrappers for C or Python, no glibc version dependency.
- Decimal arithmetic (38 significant digits precision) for rational numbers.
- IEEE 754 (15 significant digits precision) for irrational numbers.
- Intuitive context dependent user input with subscript/superscript support.
- Stateful operation for an intuitive workflow: can repeat the last operation, functions are context dependent therefore e.g. `√` or `sin` can be applied both before or after an operand.
- Logical and aesthetic layout with 20 carefully crafted themes, user configurable fonts and button shapes.
- Configurable decimal and thousands separator.
- Hackable button layout via editing config.toml.
- Proper error messages.


## Recommended fonts to install for the themes:

 - Cupertino: [SF Pro Display](https://www.cufonfonts.com/font/sf-pro-display) 
 - Redmond: [Segoe UI](https://www.cufonfonts.com/font/segoe-ui-4)
 - High Contrast: 
 - Cosmic: [Open Sans](https://github.com/googlefonts/opensans/tree/main/fonts/noto-set/ttf)
 - Monokai: [Noto Sans Math](https://github.com/notofonts/math/releases)
 - Emerald: [Adwaita Sans] or [Open Sans] or [Noto Sans Math] or [Fira Sans Math]
 - Touch: [Adwaita Sans] or [Open Sans]
 - Flat: [Lete Sans Math](https://github.com/abccsss/LeteSansMath/releases)
 - Plastic: [Adwaita Sans] or [Helvetica Neue]
 - Barbie: [Fair Prosper](https://www.dafont.com/fair-prosper.font)
 - Crystal: [Noto Sans Math] or [Fira Sans Math]
 - Cyberpunk: [Helvetica Neue] or [Comfortaa] or [Noto Sans Math] or [Fira Sans Math]
 - Matrix: [JetBrains Mono](https://github.com/JetBrains/JetBrainsMono/releases)
 - Army: [ArmyChalk](https://www.fontspace.com/armychalk-font-f9494)
 - Sandstone: [Roboto Slab](https://github.com/googlefonts/robotoslab/tree/main/fonts/otf)
 - Wolfenstein: [zilverstone eYe/FS](https://fontstruct.com/fontstructions/show/485705/zilverstone_eye_fs)

If a font is  missing, the default general system font will be used or the one you select

## Why we need a better calculator for Linux:

 Because none of the Linux calculators (KCalc, Gnome Calculator, Galculator, MATE Calculator) match the macOS or Windows calculator's user experience and I'm tired of raising issues to them which will never get fixed because it would either require too much work or it goes against their design philoshopy.
 
 A few examples (all of these problems are addressed by this calculator):
 - They are all trying to fit in into their desktop environment and therefore using the standard (Qt/GTK) buttons, radio buttons, drop-down menus etc. This does not give a good user experience in most cases. There is a reason why Apple (who is very much into UI consistency) gave a distinct look and button behaviour to their macOS calculator.
 - None of the Linux calculators above are stateful which means e.g. you cannot simply repeat the last operation by pressing equals sign (or just enter) repeatedly (as you would do on a real calculator). This is because they are all expression solvers rather than calculators.
 - KCalc, Gnome Calc, MATE Calc lets you to enter `8+++1` which will result in a `Malformed expression`. (Galculator doesn't allow expressions to be entered)
 - KCalc, MATE Calc,  solves `sqrt(-2)` resulting in `1,4142135623730950488i` and even though it is mathematically correct (so it won't be fixed) I'm pretty sure most of the users expect an error and not a complex number. By the way: you've missed that `i` at the end, haven't you?.
 - When you press a number on your keypad with Gnome Calculator, it might not get entered if the text area of the calculator was not active, so you need to reach for the mouse to click where you want to enter that number.
 - KCalc doesn't give proper error messages in most cases just a `Math error`. I think we can do better than this. There are not that many scenarios which need a separate error message.
 - Some of the Linux calculators doesn't even allow changing the font because they prefer a consistent look across all desktop applications, but I think a font that matches well with the settings menu does not necessarily a good pick for a calculator.
 - Changing colours is either limited (KCalc, galculator) or non-existent (Gnome Calc, MATE Calc). It is not possible to assign colours on a button or button group basis. Gnome Calculator somewhat mitigates this by giving different colour for the numbers and for the equals, but this is not enough. Different colours for the different function groups also help finding the function you need, but unfortunately this goes against the coherent desktop look, therefore theme support is not something the existing Linux calculators want.
 - Button layout is often not logical or aesthetically not pleasing and this is not just about making the application to look good, but also to make it easier to find the functions you need.
 - Gnome Calculator is missing useful function buttons like: `√`, `1/x`, `+/-` sure you can type in `sqrt()` but that is why it is an expression solver and not a calculator.
 - Entering sqrt or sin operation needs to come before the operand, but what if you want to apply it on the result of your previous calculation? You cannot do that and this is a very common scenario. Apple calculator does this in a smart way that allows these functions both before or after depending on the context, so it somehow always does what the user would expect.
 - Representing degree of a root or power or base of logarithm is working in an ugly and hard-to-read ASCII way most of the time and it is either incredibly cumbersome to enter an expression into these places or outright impossible.
 - Window scaling is either not possible (MATE Calc) or doesn't scale the text proportionally with the buttons (galculator) or doesn't scale the buttons at all (Gnome Calc). KCalc does it somewhat correctly, though the Deg/Rad switcher and expression display area does not scale.
 - galculator doesn't support decimal arithmetic and therefore fails the `0.1 + 0.2 - 0.3 = 0` test.
 - KCalc continuously evaluating the entered expression, which might sound like a good idea at first, but the problem is that you enter `5 + 3` and you see the result `8` on the screen. Then you enter `/2` and you would expect it will calculate `8 / 2` but instead of getting `4` you will get `6.5` because you are still editing the original expression

## Decimal Arithmetic (i128) vs Floating Point IEEE 754 (f64)

 - The problem comes from computers being binary in nature and `0.1` cannot be represented as a finite number in binary and IEEE 754 stores rounded approximations which can result in small errors.
 - `0.1 + 0.2 - 0.3` in float64 it will result in a very small number `5.55111512312578*10^(-17)` but not zero, so it is incorrect. Same problem with modulo `0.3 % 0.1` which results in `0.1` instead of zero or `1e16+1-1e16` which results in `0` instead of one, and that is quite literally a day and night difference.
 - Rounding can mitigate the small error for a while, but if you chain together many calculations it will show up sooner or later as a much bigger error.
 - The real solution is to use decimal arithmetic as long as there is no irrational number involved in the calculation e.g.: `0.1 + 1.25³ / 3` is okay, but switch to IEEE 754 as soon as irrational numbers and transcendental functions e.g. `√2 + cos(30) + log10(2) + π` come to the picture, since then we need approximation anyway and float64 gives better accuracy and speed due to hardware accelerated CPU instructions.

## Out of scope for this calculator:

- Arbitrary "infinite" precision arithmetic (Windows Calculator has it)
- Integral, derivative, lim, combinations (nCr), permutations (nPr), Fibonacci function
- Complex numbers and imaginary units
- Programmer's operations: bitshift, binary, hexadecimal calculations
- Economic and statistics calculations: mean, standard deviation, sum of squares
- Graphing calculations
- Date, Currency, Unit conversions (we want to keep this tool to be 100% offline)
- Area, perimeter, volume, surface formulas
- Physics, chemistry formulas/constants

## Usage of AI tools

- The code is 100% AI written
- Every release is tested with also AI written automated test cases and every major release is audited by AI for exploits and vulnerabilities
- There is a lot of human effort put into manually testing the software from different aspects, verifying the integration with different DEs (KDE, Gnome, MATE, Xfce, Cinnamon, Cosmic, Hyprland) and optimising to be lightweight and run well on different computers from high performance desktop (AMD 9850x3D + NVIDIA 5080) to passively cooled RaspberryPi 2 or Intel Celeron N4500
- Since this is a well confined project I believe it is possible to reach 100% security (we barely have I/O to the OS and there is no networking involved)
- Since my main issue with the existing calculators on Linux is the "it's good enough" mindset, I keep the testing to the highest standards until the application is perfectly polished

## License

GPL-3.0-only. See [LICENSE](LICENSE).
