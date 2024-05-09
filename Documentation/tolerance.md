# Tolerance value in IDS

This document summarizes what was commented on in issues [78](https://github.com/buildingSMART/IDS/issues/78) and [36](https://github.com/buildingSMART/IDS/issues/36) and the various calls.

## Numeric Data Type

IDS supports only integers and doubles. The tolerance value applies only to doubles for both ids:simpleValue and xs:resctriction.

## Tolerance

A tolerance value must always be considered for the equality of floating-point numbers, because of rounding errors.
IDS uses a fixed tolerance value (**ϵ = 1.0e⁻⁶**). The tolerance covers rounding problems when using 32-bit floating point math. Therefore, it need not be configurable: it is specific only for rounding errors, not for construction-related tolerances, in which case users need to provide an explicit range. [@aothms [issue 36 comment](https://github.com/buildingSMART/IDS/issues/36#issuecomment-1014473533)]

Six significant digits are rather pessimistic, based on the number of significant digits that can safely survive a roundtrip as 32-bit float. [@aothms [issue 78 comment](https://github.com/buildingSMART/IDS/issues/78#issuecomment-1245025134)]

If we'd use the **|a - b| < ϵ** kind of fixed tolerance, it would mean that you can't really constrain the square millimeter of a wire and that for a rail way length, you're essentially comparing without tolerance because **a + ϵ = a** in floating point math for any sufficiently large **a** and small **ϵ**. [@aothms [issue 78 comment](https://github.com/buildingSMART/IDS/issues/78#issuecomment-1245011039)]

The solution is:
**x = v implies (v - abs(v) × ϵ - ϵ) < x < (v + abs(v) × ϵ + ϵ)**
with **ϵ = 1.0e⁻⁶**
So, there is a relative component and a fixed component. [@aothms [issue 78 comment](https://github.com/buildingSMART/IDS/issues/78#issuecomment-1594479489)]

This means:

|v|lower bound<br>(v - abs(v) × ε - ε)|	upper bound<br>(v + abs(v) × ε + ε)|	absolute delta|
|---:|-------------:|-------------:|--:|
| 100000 |		99999,899999 |	100000,100001 |	0,200002 |
| 10000	 |	9999,989999	| 10000,010001 |	0,020002 |
| 1000	 |	999,998999	| 1000,001001 |	0,002002 |
| 100	   |	99,999899	| 100,000101 |	0,000202 |
| 10     |		9,999989	| 10,000011 |	2,2E-05 |
| 1	|	0,999998	| 1,000002 |	4E-06 |
| 0,1	|	0,0999989	| 0,1000011 |	2,2E-06 |
| 0,01 |		0,00999899 |	0,01000101 |	2,02E-06 |
| 0,001	|	0,000998999	| 0,001001001 |	2,002E-06 |
| 0,0001	|	0,0000989999 |	0,0001010001 |	2,0002E-06 |
| 0,00001	|	0,00000899999 |	0,00001100001 |	2,00002E-06 |
| 0,000001	|	-0,000000000001 |	0,000002000001 |	2,000002E-06 |
| 0,0000001	|	-0,0000009000001 |	0,0000011000001 |	2,0000002E-06 |
| 0	|	-0,000001 |	0,000001 |	2,0E-06 |
| -0,0000001	|	-0,0000011000001 |	0,0000009000001 |	2,0000002E-06 |
| -0,000001	|	-0,0000020000 |	0,000000000001 |	2,000002E-06 |
| -0,00001	|	-0,0000110000 |	-0,00000899999 |	2,00002E-06 |
| -0,0001	|	-0,0001010001 |	-0,0000989999 |	2,0002E-06 |
| -0,001	|	-0,0010010010 |	-0,000998999 |	2,002E-06 |
| -0,01	|	-0,0100010100 |	-0,00999899 |	2,02E-06 |
| -0,1	|	-0,1000011000 |	-0,0999989 |	2,2E-06 |
| -1,0	|	-1,0000020000 |	-0,999998 |	4E-06 |
| -10,0	|	-10,0000110000 |	-9,999989 |	2,2E-05 |
| -100,0	|	-100,0001010000 |	-99,999899 |	0,000202 |
| -1000,0	|	-1000,0010010000 |	-999,998999 |	0,002002 |
| -10000,0	|	-10000,0100010000 |	-9999,989999 |	0,020002 |
| -100000,0	|	-100000,1000010000 |	-99999,899999 |	0,200002 |
| -1000000,0	|	-1000001,0000010000 |	-999998,999999 |	2,000002 |

# Tolerance and range

Adding tolerance to Range checks opens up all kinds of strange edge cases: `41.999958d satisfies being in the range 42-50 inclusive, but also satisfies being < 42 exclusive` [@andyward [issue 78 comment](https://github.com/buildingSMART/IDS/issues/78#issuecomment-2012281379)]

Exclusive ranges should be shrunk, not stretched, to match the semantic intent of the requirement. So > 0 actually becomes > 1.0e⁻⁶; while Inclusive ranges are stretched >= 0 becomes >= -1.0e⁻⁶. If a numeric RangeConstraint's spread is < [agreed-calculation] the Audit-tool should raise a warning. [@andyward [issue 78 comment](https://github.com/buildingSMART/IDS/issues/78#issuecomment-2012672961)]
