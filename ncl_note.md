[TOC]

# ncl note

## Symbols
- ;   &emsp;      begins a comment
- /;...;/  &emsp;  block comment
- =     &emsp;    assignment
- :=    &emsp;    reassignment
- @     &emsp;    create or reference attributes
- !     &emsp;    create or reference named dimensions
- &     &emsp;    create or reference a coordinate
- {...} &emsp;    used for coordinate subscripting
- $     &emsp;    enclose strings when importing/exporting variables via **addfile**
- [...] &emsp;   subscript variables of type **list**
- (/.../) &emsp;  construct an array
- :     &emsp;    used in array syntax
- |     &emsp;    used as a separator for named dimensions
- \     &emsp;    continue statement for spanning multiple lines
- ::    &emsp;    used as separator when calling external codes
- ->    &emsp;    used for variable input/output with supported data formats
- =>    &emsp;    used to access netCDF-4/HDF5 groups

## data types
- numeric: 
  double (64 bit),float (32 bit),long (32 or 64 bit),integer (32 bit),short(8 bit)
- non-numeric: 
  string, character, graphic, file, logical, list

## expressions

- algebraic operators:
  same as other languages
- logical:
  - .lt.  &emsp;  less than
  - .le.  &emsp;  less than or equal to 
  - .gt.  &emsp;  greater than
  - .ne.  &emsp;  not equal to
  - .eq.  &emsp;  equal to
  - .and. &emsp;  and
  - .or.  &emsp;  or
  - .xor. &emsp;  exclusive or
  - .not. &emsp;  not

## loops
- do-end do 
  ```
  do n=start,end,optional_stride
        [statement(s)]
  end do
  ```
- do while-end do
  ```
  do while (scalar_logical_expression)
        [statement(s)]
  end do
  ```
- also can use **break** and **continue**

## if statements
```ncl
if (scalar_logical_expression) then
    [statement(s)]
else
    [statement(s)]
end if
```
