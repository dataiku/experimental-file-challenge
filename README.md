# Experimental file challenge
Dataiku Developer Challenge (ecosystem)

## Overview
A Dataiku Experimental File (DEF) stores data in a way similar to Excel or CSV files. Like Excel files, DEF files are binary files with a specific structure. DEF files have a specified number of columns, each with a name and a type (either String, Integer or Floating-point numbers) and can have any number of rows. Like CSV files, NULL values are not supported so each cell contains an actual value.

A DEF file starts with a header block containing the metadata of the files, including the total number of rows, the description of each column.
Unlike CSV files, in DEF files data are stored in a columnar layout, meaning that all values for a particular column are stored together.

Here is how a DEF file is organized:
```
| Header …. | column values | column values | … | column values |
```

As an example, the following table is stored in the following ways:
```
 ---------------------------------
| Movie           | Rating | Year |
|-----------------|--------|------|
| The Dark Knight |    9.0 | 2008 |
| Pulp Fiction    |    8.9 | 1994 |
| Fight Club      |    8.8 | 1999 |
| Inception       |    8.7 | 2010 |
 ---------------------------------
```

```
Header            : Movie (string), Rating (float), Year (integer)
Movie column data : The Dark Knight, Pulp Fiction, Fight Club, Inception
Rating column data: 9.0, 8.9, 8.8, 8.7
Year column data  : 2008, 1994, 1999, 2010
```

## Specifications
Here is the full specification of Dataiku Experimental Files:

**Header block**
```
 --------------------------------------------------------------
| Offset   | Content                                | Length   |
|----------|----------------------------------------|----------|
| 0x0000   | Signature (44 4b 55 45 46 48 = DKUEFH) | 6 bytes  |
| 0x0006   | Version (00 01)                        | 2 bytes  |
| 0x0008   | Header length                          | 4 bytes  |
| 0x000C   | Number of rows                         | 8 bytes  |
| 0x0014   | Number of columns                      | 4 bytes  |
| 0x0018   | Column description (for 1st column)    | variable | 
| variable | Column description (for 2nd column)    | variable |
| ...      | ...                                    | ...      |
| variable | Column description (for last column)   | variable |
| variable | Padding (1)                            | variable |
 -------------------------------------------------------------- 

(1) If the length of the header is not a multiple of 8, some padding is
added so that the first column data block is aligned on an 8 bytes boundary.
```

**Header column description**
```
 ------------------------------------------------------------ 
| Offset   | Content                              | Length   |
|----------|--------------------------------------|----------|
| 0x0000   | Column type (1)                      | 2 bytes  |
| 0x0001   | Column name (2)                      | variable |
| variable | Column data block offset (4)         | 8 bytes  |
 ------------------------------------------------------------ 

(1) Possible values are:
      01: String (utf8 encoded, zero-terminated)
      02: 64 bit integer number
      03: 64 bit floating number
(2) Column names are encoded in utf-8 and terminated with a zero
(3) Position, relative to the beginning of the file where values for this column are stored. 
```
 
**Column data block**
```
 -------------------------------------------------------------- 
| Offset   | Content                                | Length   |
|----------|----------------------------------------|----------|
| 0x0000   | Signature (44 4b 55 43 4f 4c = DKUCOL) | 6 bytes  |
| 0x0006   | Unused / padding                       | 2 bytes  |
| 0x0008   | Column values                          | variable |
| variable | Padding (1)                            | variable |
 -------------------------------------------------------------- 

(1) If the length of the column data block is not a multiple of 8, some padding is added so that the next column data block is aligned on an 8 bytes boundary. 
Challenge
```

## Mission

Your mission is to write a program that takes as input a DEF file and print it on the console in a human readable format.

For example, given the following movie.def file, 
```
444b 5545 4648 0001 0000 0048 0000 0000
0000 0004 0000 0003 0001 4d6f 7669 6500
0000 0000 0000 0048 0003 5261 7469 6e67
0000 0000 0000 0000 8800 0259 6561 7200
0000 0000 0000 00b0 444b 5543 4f4c 0000
5468 6520 4461 726b 204b 6e69 6768 7400
5075 6c70 2046 6963 7469 6f6e 0046 6967
6874 2043 6c75 6200 496e 6365 7074 696f
6e00 0000 0000 0000 444b 5543 4f4c 0000
4022 0000 0000 0000 4021 cccc cccc cccd
4021 9999 9999 999a 4021 6666 6666 6666
444b 5543 4f4c 0000 0000 0000 0000 07d8
0000 0000 0000 07ca 0000 0000 0000 07cf
0000 0000 0000 07da
```

Your program should print something similar to:
```
$>def-dump movie.def
 ---------------------------------
| Movie           | Rating | Year |
|-----------------|--------|------|
| The Dark Knight |    9.0 | 2008 |
| Pulp Fiction    |    8.9 | 1994 |
| Fight Club      |    8.8 | 1999 |
| Inception       |    8.7 | 2010 |
 ---------------------------------
```
