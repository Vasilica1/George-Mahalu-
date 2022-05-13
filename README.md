$MOD51
RS EQU P3.7
EN EQU P3.6
str0 equ 30h
str1 equ 31h
str2 equ 32h
str3 equ 33h
num equ 34h
opnd1_1 equ 35h
opnd1_2 equ 36h
opnd1_3 equ 37h
opnd2_1 equ 38h
opnd2_2 equ 39h
opnd2_3 equ 3Ah
opt equ 3Bh
SIGN EQU 3CH
number1 equ 3dh
number2 equ 3eh

 mov p1,#0ffh
 mov str3,#0
 MOV SIGN,#' '
 mov number1,#0
 mov number2,#0
 LCALL KEYSCAN       ;Read the first operand
 MOV NUM,R7
 LCALL convKey     
 MOV OPND1_1,A
 LCALL WRITE_CHAR
 clr c
 subb a,#48
 mov number1,a
 
 LCALL KEYSCAN       ;Read the first operand
 MOV NUM,R7
 LCALL convKey     
 
 CJNE A,#'+',NXX1
 JMP OPERATOR
 NXX1:CJNE A,#'-',NXX2
 JMP OPERATOR
 NXX2:CJNE A,#'*',NXX3
 JMP OPERATOR
 NXX3:CJNE A,#'/',NXX4
 JMP OPERATOR
 NXX4:CJNE A,#'!',NXX5
 JMP NOTOPERATION
 NXX5:
 LCALL WRITE_CHAR
 CLR C
 SUBB A,#48
 MOV OPND1_2,A

 MOV A,NUMBER1
 MOV B,#10
 MUL AB
 ADD A,OPND1_2
 MOV NUMBER1,A
 
    LCALL KEYSCAN      
 MOV NUM,R7
 LCALL convKey     
 
 CJNE A,#'+',NXY1
 JMP OPERATOR
NXY1:CJNE A,#'-',NXY2
 JMP OPERATOR
NXY2:CJNE A,#'*',NXY3
 JMP OPERATOR
NXY3:CJNE A,#'/',NXY4
 JMP OPERATOR
NXY4:CJNE A,#'!',NXY5
 JMP NOTOPERATION
NXY5:
LCALL WRITE_CHAR
CLR C
SUBB A,#48
MOV OPND1_3,A

MOV A,NUMBER1
MOV B,#10
MUL AB
ADD A,OPND1_3
MOV NUMBER1,A
 
  
LCALL KEYSCAN 
MOV NUM,R7
LCALL convKey
OPERATOR: 
MOV OPT,A
LCALL WRITE_CHAR
CJNE A,#'!',get2ndOP
jmp notoperation
get2ndOP: 
LCALL KEYSCAN     
MOV NUM,R7
LCALL convKey     
MOV OPND2_1,A
LCALL WRITE_CHAR
clr c
subb a,#48
mov number2,a
 
LCALL KEYSCAN       
MOV NUM,R7
LCALL convKey     
CJNE A,#'=',NXZ1
JMP COMPUTE



NXZ1:
 LCALL WRITE_CHAR
 CLR C
 SUBB A,#48
 MOV OPND2_2,A
 
 MOV A,NUMBER2
 MOV B,#10
 MUL AB
 ADD A,OPND2_2
 MOV NUMBER2,A
 
    LCALL KEYSCAN       
 MOV NUM,R7
 LCALL convKey     
 
 CJNE A,#'=',NXS1
 JMP COMPUTE

NXS1:
    LCALL WRITE_CHAR
 CLR C
 SUBB A,#48
 MOV OPND2_3,A
 
 MOV A,NUMBER2
 MOV B,#10
 MUL AB
 ADD A,OPND2_3
 MOV NUMBER2,A
 
 MOV A,#'='
COMPUTE:
 LCALL WRITE_CHAR

 
    MOV A,OPT 
  
    CJNE A,#'+',NXOP1
 MOV A,NUMBER1
 ORL A,NUMBER2
 ACALL CONVERT
 ACALL SHOW
 JMP ENDX
NXOP1:
 CJNE A,#'-',NXOP2
 MOV A,NUMBER1
 MOV B,NUMBER2
 ORL A,B
 CPL A
 ACALL CONVERT
 ACALL SHOW
 JMP ENDX
NXOP2:
 CJNE A,#'*',NXOP3
 MOV A,NUMBER1
 MOV B,NUMBER2
 ANL A,B
 ACALL CONVERT
 ACALL SHOW
 JMP ENDX
NXOP3:
 CJNE A,#'/',NXOP3
 MOV A,NUMBER1
 MOV B,NUMBER2
 ANL A,B
 CPL A
 ACALL CONVERT
 ACALL SHOW
 JMP ENDX
NOTOPERATION:
 MOV A,#'!'
 LCALL WRITE_CHAR
 MOV A,#'='
 LCALL WRITE_CHAR
 MOV A,NUMBER1
 CPL A
 ACALL CONVERT
 ACALL SHOW
 

ENDX:JMP $

CONVERT:
POS:
 MOV B,#10
 DIV AB
    PUSH ACC
    MOV A,B
 ADD A,#48
    MOV STR2,A
 POP ACC
 MOV B,#10
 DIV AB
 
 PUSH ACC
    MOV A,B
 ADD A,#48
    MOV STR1,A
    
 POP ACC
 ADD A,#48
 MOV STR0,A
 RET

SHOW:
 MOV A,STR0
 CALL WRITE_CHAR
    MOV A,STR1
 CALL WRITE_CHAR
    MOV A,STR2
 CALL WRITE_CHAR
    RET

SHOWPOS:
 MOV A,STR1
 CJNE A,#'0',showit1
 JMP ONEDIG1
showit1:
    CALL WRITE_CHAR
ONEDIG1:
 MOV A,STR2
 CALL WRITE_CHAR
 RET

WRITE_CMD:
 CLR RS
 MOV P2,A
 SETB EN
 CLR EN
 RET

WRITE_CHAR:
 SETB RS
 MOV P2,A
 SETB EN
 CLR EN
 RET

INITIALIZE_LCD:
 MOV A,#38H  
 LCALL WRITE_CMD
 MOV A,#0CH  
 LCALL WRITE_CMD
 MOV A,#06H  
 LCALL WRITE_CMD
 RET

GOTO_ADDRESS:
 ADD A,#80H
 LCALL WRITE_CMD
 RET

CLEAR_LCD:

 MOV A,#01H
 LCALL WRITE_CMD
 RET

mydelay:
 mov r0,#255
 djnz r0,$
 ret


keyscan:
PUSH ACC
MOV A, R0
PUSH ACC
MOV A, R1
PUSH ACC
MOV A, R3
PUSH ACC
MOV P1, #00FH
waitkeyuploop:
MOV A, P1
ANL A, #00FH
XRL A, #00FH
JNZ waitkeyuploop
 MOV P1, #00FH
anykeyloop:
MOV A, P1
ANL A, #00FH
XRL A, #00FH
JZ anykeyloop
MOV R0, A
XRL A, #00FH
MOV R3, A
call lineclear
MOV P1, #0EFH
MOV R1, #010H
NOP
MOV A, P1
ANL A, #00FH
XRL A, R3
MOV R2,#000H
JZ scanmatch

call lineclear
 MOV P1, #0DFH
MOV R1, #020H
NOP
MOV A, P1
ANL A, #00FH
XRL A, R3
MOV R2, #001H
JZ scanmatch
call lineclear
MOV P1, #0BFH
 MOV R1, #040H
NOP
MOV A, P1
ANL A, #00FH
XRL A, R3
MOV R2, #002H
JZ scanmatch

call lineclear
 MOV P1, #07FH
 MOV R1, #080H
 NOP
 MOV A, P1
 ANL A, #00FH
 XRL A, R3
MOV R2, #003H
JZ scanmatch
MOV P1, #00FH
JMP keyscan
scanmatch:
MOV A, R1
ORL A, R0   ; combine into scancode

MOV R7, A ; R7 contains value in (row,col) format
keyscanend:
POP ACC
MOV R3, A
POP ACC
MOV R1, A
POP ACC
MOV R0, A
POP ACC
RET

lineclear:
        MOV P1, #0FFH
lineclearloop:
        MOV A, P1
        XRL A, #0FFH
        JNZ lineclearloop
        RET

convKey:
 mov A,num
 cjne a,#24,nk1
 mov a,#'!'
 ret
nk1:cjne a,#20,nk2
 mov a,#'0'
 ret
nk2:cjne a,#18,nk3
 mov a,#'='
 ret
nk3:cjne a,#17,nk4
 mov a,#'/'
 ret
nk4:cjne a,#40,nk5
 mov a,#'1'
 ret
nk5:cjne a,#36,nk6
 mov a,#'2'
 ret
nk6:cjne a,#34,nk7
 mov a,#'3'
 ret
nk7:cjne a,#33,nk8
 mov a,#'*'
 ret
nk8:cjne a,#72,nk9
 mov a,#'4'
 ret
nk9:cjne a,#68,nk10
 mov a,#'5'
 ret
nk10:cjne a,#66,nk11
 mov a,#'6'
 ret
nk11:cjne a,#65,nk12
 mov a,#'-'
 ret
nk12:cjne a,#136,nk13
 mov a,#'7'
 ret
nk13:cjne a,#132,nk14
 mov a,#'8'
 ret
nk14:cjne a,#130,nk15
 mov a,#'9'
 ret
nk15:cjne a,#129,nkX
 mov a,#'+'
 ret
nkX:mov a,#'X'
 ret
END
