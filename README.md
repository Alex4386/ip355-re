# MOIMSTONE IP355 - Reverse Engineering
"프로그래머 아님" + "해커 아님" + "정보보안 알못" 의 모임스톤 IP355 역공학 "유사" 라이트업

## 하게 된 이유
벨소리 바꾸고 전역하면 **[@Stella-IT](https://github.com/Stella-IT)** PBX 에 묶어서 쓰려고

## Spec
해당 기종은 IP355 H/W Revision 1.1 을 기준으로 작성됨.

* AP: Broadcom BCM1190KQMG
  - MIPS32 with DSP Instruction Set
  - UART + JTAG (IEEE 1149.1)
* Flash: Winbond 25Q32FVSIG (32K Flash)
* RAM: Winbond DRAM (DDR2?) 
* LCD: LF23020A, (Vendor unspecified)

데이터시트는 알아서 찾아보십쇼.

## Flash Dump
IP355 Firmware 1.10.012_d _(Analyzed with Binwalk)_  
![image](https://user-images.githubusercontent.com/27724108/189259794-9a21e04c-bf10-425f-99c9-3be5fb4b0395.png)

Estimated build date: 2012/02/20  

## Operating System
WindRiver Systems **VxWorks** 5.5.1 with Moimstone customization.  
![image](https://user-images.githubusercontent.com/27724108/189260078-55326dc4-95d2-4398-a823-31b474416deb.png)



