# MOIMSTONE IP355 - Reverse Engineering
<img src="https://user-images.githubusercontent.com/27724108/189268814-a558de9a-d8ed-4d26-b40a-c868591d8ba5.png" width="200" />
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
![image](https://user-images.githubusercontent.com/27724108/189265833-03215a33-2d7b-46c1-a2a2-3226029f98cc.png)  

Estimated build date: 2012/02/20  
**꼭 binwalk는 최신버전 쓰세요....**  

우선 맨 앞에 있는 로더 같은걸 먼저 뜯어 보자.

### 환경, 환경을 알자.
아까 칩셋 정보를 통해 MIPS32 with DSP Instruction 인걸 확인했다. 문제는 조금 더 상세정보를 알아야 하는데, 이건 binwalk 의 `--disasm` 플래그를 이용해보자.
![image](https://user-images.githubusercontent.com/27724108/189265719-64d09ab5-9127-43c6-902a-f534615c290b.png)

Binwalk 최고라고 외치고 Big Endian 이라는 걸 알았다.

### OS Loader
WindRiver Systems **VxWorks** 5.5.1 with Moimstone customization. _(Analyzed with Binwalk)_    
![image](https://user-images.githubusercontent.com/27724108/189260078-55326dc4-95d2-4398-a823-31b474416deb.png)

진짜 별거 없다. 그럼 안에 아까 위에서 봤던 LZMA를 뜯어봐야겠다.  
![image](https://user-images.githubusercontent.com/27724108/189267078-dceecf2f-d8a9-49e2-9871-ab8f5d372ac0.png)

얼레? LZMA 파일 타입을 인식할 수 없댄다. 아까 binwalk 아웃풋을 한번 다시 봐보자.  
![image](https://user-images.githubusercontent.com/27724108/189265833-03215a33-2d7b-46c1-a2a2-3226029f98cc.png)

체인로딩 될 코드가 시작되는 메모리 주소 치고 영 깔끔한 주소가 아니다. 0x400AC? 아무리 모임스톤쪽 개발자가 펌웨어 만들면서 "에이씨" 라고 외치고 싶다고 해도 메모리 주소를 이렇게 정하지는 않았을꺼라는 합리적인 추론을 해볼 수 있다.  

그럼 과연 뭘까? 이때 뜬금포로 데이터사이언스가 튀어나온다.  
엔트로피를 써서 데이터가 어떤지 꼴을 한번 봐보자.  

`binwalk` 의 `-E` 플래그를 줘서 엔트로피를 확인해 볼 수 있다.  
![image](https://user-images.githubusercontent.com/27724108/189267360-0745ec50-5e0b-4fd9-9825-0c9651748dd9.png)  

그래프를 보니 합리적인 의심은 확신으로 바뀌어 간다. 0x40000, 0x40000을 보자.  
![image](https://user-images.githubusercontent.com/27724108/189267473-8f7008b4-7d69-4f20-ad8e-57c1847d8e26.png)

BFS 헤더? 파일이름??? 이건 대충봐도 파일시스템 구조 같아보인다.  
![image](https://user-images.githubusercontent.com/27724108/189267827-da23aa4b-40f4-4f45-b14c-1f497a51d296.png)

그럼 로더쪽 펌웨어에서 파일시스템으로 grep 때려서 파일시스템 로딩하는 쪽 로그 string 이 있는지 찾아보자.  
![image](https://user-images.githubusercontent.com/27724108/189268128-b190f8e0-60ae-4e85-b6ff-bb3aa1388c36.png)

b...cm...fs? 이게 뭐지? **B**road**c**o**m** **F**ile**S**ystem 인가?

### Broadcom FileSystem을 찾아서. (Part 1. Google神)
우선 이왕이면 내가 직접 구현하기는 싫으니 BFS (BCMFS) 관련 자료가 인터넷에 있는지 구글신께 제사를 지내봤다.  
![image](https://user-images.githubusercontent.com/27724108/189506484-be34639f-6887-4bda-bae7-3810eba179e5.png)  

DPDK? 이건 kernel 에서 패킷 다이렉트로 가져올 때 쓰는 거잖아...? 이거 뭐임? Broadcom FlexSparc???  
![image](https://user-images.githubusercontent.com/27724108/189506510-22b4fc3c-4eb5-4ea4-b35b-4644818325b1.png)  

구글신이 답을 주긴 커녕 그냥 오히려 신탁이라는 엿을 줬다.  
하....  

![image](https://user-images.githubusercontent.com/27724108/189506578-3008840d-433a-4c7c-ba66-728402138059.png)  

#### Broadcom FileSystem을 찾아서. (Part 2. 모든 것을 Micro$oft 에 맡긴 "김명자 낙지마당")
그래도 희망을 저버리고 싶진 않다. 적어도 난 [ghidra](https://github.com/NationalSecurityAgency/ghidra-sre) 에서 MIPS32 어셈블리 일일이 읽으면서 하고 싶진 않기 때문이다....  

![image](https://user-images.githubusercontent.com/27724108/189506644-88d292c1-c305-455f-90a5-08a0982cf193.png)  
  
즈에발 이라면서 bcmfs 와 BFS 키워드로 "모든것을 Micro$oft에 맡긴 김명자 낙지마당" 에게 물어봤다.  
![image](https://user-images.githubusercontent.com/27724108/189506673-fc850f58-4583-44c3-9268-f2393c98944d.png)  

어? WRT? 이거 공유기 펌웨어 그런거 아님?  
![image](https://user-images.githubusercontent.com/27724108/189506741-376bfb1e-eb1c-4249-82e5-ec9a8df4e067.png)

생각해 보니 임베디드 해킹이면 OpenWRT 레포를 볼 생각을 안해봤다는 것도 나 자신이 🅱️🆎🅾️ 라는 걸 증명하는 것 같았다.

결론은
나중에 이어서...
