# CBT267
Converted to GitHub via [cbt2git](https://github.com/wizardofzos/cbt2git)

This is still a work in progress. 
Due to amazing work by Alison Zhang and Jake Choi repos are no longer deleted.

```
//***FILE 267 is the HETUTL utility from Leland Lucius.  This is    *   FILE 267
//*           a program which runs under any MVS system, and which  *   FILE 267
//*           reads a tape, converting it either into AWS format    *   FILE 267
//*           or into HET (Hercules Emulated Tape) format.          *   FILE 267
//*                                                                 *   FILE 267
//*           HET format (which was invented by Leland Lucius) is   *   FILE 267
//*           a compressed variation of an AWS tape, and which is   *   FILE 267
//*           directly usable as a tape, by a Hercules S/390        *   FILE 267
//*           emulator, running on a PC.  HET tape format is not    *   FILE 267
//*           usable by a P/390, but the straight AWS-format        *   FILE 267
//*           that HETUTL is capable of producing, is usable by     *   FILE 267
//*           a P/390.  (See also the VTT2DISK utility on File      *   FILE 267
//*           533, which produces AWS-format tape files.  Also      *   FILE 267
//*           see the AWSUTIL utility from File 477.)               *   FILE 267
//*                                                                 *   FILE 267
//*           The compression routines used by HETUTL are either    *   FILE 267
//*           the ZLIB compression or the BZLIB compression         *   FILE 267
//*           routines.  We have to thank Thomas David Rivers       *   FILE 267
//*           of Dignus LLC, for having supplied the Systems/C      *   FILE 267
//*           compiler, which was used to convert the C-Language    *   FILE 267
//*           code for these compression routines, into Assembler   *   FILE 267
//*           Language source code.                                 *   FILE 267
//*                                                                 *   FILE 267
//*           A load module library containing the HETUTL load      *   FILE 267
//*           module, has been included in this file, as member     *   FILE 267
//*           $LOADLIB.                                             *   FILE 267
//*                                                                 *   FILE 267
//*           email:  "Leland Lucius" <yam@homerow.net>             *   FILE 267
//*                                                                 *   FILE 267
//*    - - - - - - - - - - - - - - - - - - - - - - - - - - - - -    *   FILE 267
//*                                                                 *   FILE 267
//*     HETUTL   TITLE    'Convert tape to HET format'              *   FILE 267
//*                                                                 *   FILE 267
//*     Function : Convert tapes to Hercules Emulated Tape          *   FILE 267
//*                format.                                          *   FILE 267
//*                                                                 *   FILE 267
//*     Amode    : Must run in 31-bit addressing mode               *   FILE 267
//*                                                                 *   FILE 267
//*     Rmode    : Must reside below the line                       *   FILE 267
//*                                                                 *   FILE 267
//*     Auth     : None (recently fixed not to need authorization)  *   FILE 267
//*                                                                 *   FILE 267
//*     Return   : All errors are indicated with a message          *   FILE 267
//*                to SYSPRINT and a non-zero condition             *   FILE 267
//*                code:                                            *   FILE 267
//*                                                                 *   FILE 267
//*     Descript : This program accepts an IBM format tape          *   FILE 267
//*                and converts it to an HET format file.           *   FILE 267
//*                Output compression, input block sizes of         *   FILE 267
//*                up to 65535 bytes, and options to create         *   FILE 267
//*                files compatible with the AWSTAPE format         *   FILE 267
//*                are supported.                                   *   FILE 267
//*                                                                 *   FILE 267
//*     Execparm : Specify "ABEND" to force the program to          *   FILE 267
//*                abend instead of return a condition code         *   FILE 267
//*                in the event of an error.                        *   FILE 267
//*                                                                 *   FILE 267
//*     Parms    : Blank delimited keyword/value pairs.             *   FILE 267
//*                                                                 *   FILE 267
//*           INPUT_DDNAME <x>                                      *   FILE 267
//*               DD name of input data set.                        *   FILE 267
//*                                                                 *   FILE 267
//*               Default: SYSUT1                                   *   FILE 267
//*                                                                 *   FILE 267
//*           OUTPUT_DDNAME <x>                                     *   FILE 267
//*               DD name of output data set.                       *   FILE 267
//*                                                                 *   FILE 267
//*               Default: SYSUT2                                   *   FILE 267
//*                                                                 *   FILE 267
//*           CHUNK_SIZE <n>                                        *   FILE 267
//*               Physical output blocks will be broken             *   FILE 267
//*               into <n> sized chunks.  Setting this              *   FILE 267
//*               parameter to 4096 and turning off                 *   FILE 267
//*               compression will create output                    *   FILE 267
//*               compatible with the AWSTAPE format.               *   FILE 267
//*                                                                 *   FILE 267
//*               Minimum: 4096                                     *   FILE 267
//*               Maximum: 65535                                    *   FILE 267
//*               Default: 65535                                    *   FILE 267
//*                                                                 *   FILE 267
//*           COMPRESSION_METHOD <0|1|2>                            *   FILE 267
//*               Specifies whether compression should              *   FILE 267
//*               be performed and which method to use.             *   FILE 267
//*               See note above about AWSTAPE format               *   FILE 267
//*               compatiblity.                                     *   FILE 267
//*                                                                 *   FILE 267
//*               Values:  0 - No compression                       *   FILE 267
//*                        1 - ZLIB compression                     *   FILE 267
//*                        2 - BZLIB compression                    *   FILE 267
//*               Default: 1 - ZLIB compression                     *   FILE 267
//*                                                                 *   FILE 267
//*           COMPRESSION_LEVEL <n>                                 *   FILE 267
//*               Level of compression performed.                   *   FILE 267
//*               Setting <n> to lower values decreases             *   FILE 267
//*               compression effectiveness, but will               *   FILE 267
//*               improve run times.                                *   FILE 267
//*                                                                 *   FILE 267
//*               Minimum: 1 - Fastest compression                  *   FILE 267
//*               Maximum: 9 - Best compression                     *   FILE 267
//*               Default: 4 - Middle of the road                   *   FILE 267
//*                                                                 *   FILE 267
//*           VERIFY_COMPRESSION <YES|NO>                           *   FILE 267
//*               Decompress data after compression and             *   FILE 267
//*               compare against input data.                       *   FILE 267
//*                                                                 *   FILE 267
//*               Values:  YES - Verify compression                 *   FILE 267
//*                        NO  - Bypass verification                *   FILE 267
//*               Default: NO  - Bypass verification                *   FILE 267
//*                                                                 *   FILE 267
//*           CHECK_VOLSER <YES|NO>                                 *   FILE 267
//*               Verify volser from VOL1 against the               *   FILE 267
//*               one requested.                                    *   FILE 267
//*                                                                 *   FILE 267
//*               Values:  YES - Check the volsers                  *   FILE 267
//*                        NO  - Bypass check                       *   FILE 267
//*               Default: NO  - Bypass check                       *   FILE 267
//*                                                                 *   FILE 267
//*           CLEAR_IDRC_INDICATOR <YES|NO>                         *   FILE 267
//*               Clear the IDRC indicator in the VOL1              *   FILE 267
//*               record.                                           *   FILE 267
//*                                                                 *   FILE 267
//*               Values:  YES - Clears IDRC indicator in VOL1      *   FILE 267
//*                        NO  - Ignore the indicator               *   FILE 267
//*               Default: NO  - Ignore the indicator               *   FILE 267
//*                                                                 *   FILE 267
//*           DISPLAY_ENDING_STATUS <YES|NO>                        *   FILE 267
//*               Display the sense information from                *   FILE 267
//*               the last read.                                    *   FILE 267
//*                                                                 *   FILE 267
//*               Values:  YES - Display the info                   *   FILE 267
//*                        NO  - Bypass display                     *   FILE 267
//*               Default: NO  - Bypass display                     *   FILE 267
//*                                                                 *   FILE 267
//*           ROUTE_CODE <n>                                        *   FILE 267
//*               Specifies the route code to use when              *   FILE 267
//*               issuing WTORs.  Multiple route codes              *   FILE 267
//*               may be specifed by including as many              *   FILE 267
//*               ROUTE_CODE statements as needed.                  *   FILE 267
//*                                                                 *   FILE 267
//*               Minimum: 1                                        *   FILE 267
//*               Maximum: 128                                      *   FILE 267
//*               Default: 3 (Tape Pool)                            *   FILE 267
//*                                                                 *   FILE 267
//*  JCL : //HETUTL   EXEC PGM=HETUTL,REGION=4M<,PARM=ABEND>        *   FILE 267
//*        //STEPLIB  DD DISP=SHR,DSN=<(authorized) load library>   *   FILE 267
//*        //INDD     DD DISP=SHR,DSN=<input tape>,LABEL=(1,SL)     *   FILE 267
//*        //OUTDD    DD DISP=(,CATLG),DSN=<output data set>,       *   FILE 267
//*        //            SPACE=(CYL,(10,10)),UNIT=SYSALLDA          *   FILE 267
//*        //SYSPRINT DD SYSOUT=*                                   *   FILE 267
//*        //SYSIN    DD *                                          *   FILE 267
//*        INPUT_DDNAME        INDD                                 *   FILE 267
//*        INPUT_DDNAME        OUTDD                                *   FILE 267
//*        COMPRESSION_LEVEL   9                                    *   FILE 267
//*        /*                                                       *   FILE 267
//*                                                                 *   FILE 267
//*    Notes  : The input data set must be a tape dataset.          *   FILE 267
//*                                                                 *   FILE 267
//*             The output data set may reside on DASD or           *   FILE 267
//*             TAPE and will be forced to use                      *   FILE 267
//*             DCB=(RECFM=U,LRECL=0,BLKSIZE=32760).                *   FILE 267
//*                                                                 *   FILE 267
//*             The EXCP abnormal end appendage is no               *   FILE 267
//*             longer needed so the code has been removed.         *   FILE 267
//*             This means that HETUTL no longer has any            *   FILE 267
//*             APF requirements.                                   *   FILE 267
//*                                                                 *   FILE 267
//*             Developed on OS/390 2.8/10, so compatibility        *   FILE 267
//*             with other or older/newer OSes is unknown.          *   FILE 267
//*                                                                 *   FILE 267
//*    Thanks : Special thanks go out to Dignus, LLC for            *   FILE 267
//*             providing a short term license of their             *   FILE 267
//*             Systems/C compiler.  Using their compiler           *   FILE 267
//*             removes any run-time library limitations            *   FILE 267
//*             and/or restrictions.  Thanks Dave!                  *   FILE 267
//*                                                                 *   FILE 267
```
