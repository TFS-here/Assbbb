.MODEL SMALL
.STACK 100H

.DATA
    MSG1 DB 'Enter first hex number (NUM1, 4 digits): $'
    MSG2 DB 0DH,0AH,'Enter second hex number (NUM2, 4 digits): $'

    NUM1L DB ?
    NUM1H DB ?
    NUM2L DB ?
    NUM2H DB ?
    RESL  DB ?
    RESH  DB ?

.CODE
MAIN PROC FAR
    MOV AX, @DATA
    MOV DS, AX

    ; ---- INPUT NUM1 ----
    LEA DX, MSG1
    MOV AH, 09H
    INT 21H
    CALL READ_HEX16
    MOV NUM1L, BL
    MOV NUM1H, BH

    ; ---- INPUT NUM2 ----
    LEA DX, MSG2
    MOV AH, 09H
    INT 21H
    CALL READ_HEX16
    MOV NUM2L, BL
    MOV NUM2H, BH

    ; ---- CALCULATION ----
    ; RESULT = (NUM1 + NUM2) - (NUM2 - NUM1)

    MOV AL, NUM1L
    MOV AH, NUM1H
    ADD AL, NUM2L
    ADC AH, NUM2H
    PUSH AX                ; (NUM1+NUM2)

    MOV AL, NUM2L
    MOV AH, NUM2H
    SUB AL, NUM1L
    SBB AH, NUM1H
    MOV BX, AX             ; (NUM2-NUM1)

    POP AX
    SUB AL, BL
    SBB AH, BH

    MOV RESL, AL
    MOV RESH, AH

    ; ---- Exit ----
    MOV AH, 4CH
    INT 21H
MAIN ENDP


; =====================================================
; READ_HEX16 – read 4 hex characters → return BX (16-bit)
; =====================================================
READ_HEX16 PROC
    PUSH AX
    PUSH CX
    PUSH DX

    MOV BX, 0       ; clear BX
    MOV CX, 4       ; read 4 hex digits

NEXT_DIGIT:
    MOV AH, 01H     ; read char
    INT 21H
    CMP AL, '9'
    JLE DIGIT_DEC
    CMP AL, 'F'
    JLE DIGIT_HEX
    SUB AL, 20H     ; make lowercase → uppercase
DIGIT_HEX:
    SUB AL, 'A'-10
    JMP DIGIT_READY

DIGIT_DEC:
    SUB AL, '0'

DIGIT_READY:
    SHL BX, 4
    OR BL, AL
    LOOP NEXT_DIGIT

    POP DX
    POP CX
    POP AX
    RET
READ_HEX16 ENDP

END MAIN
