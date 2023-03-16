# BCMFS 삽질

## Broadcom FileSystem을 찾아서. (Part 1. Google神)
우선 이왕이면 내가 직접 구현하기는 싫으니 BFS (BCMFS) 관련 자료가 인터넷에 있는지 구글신께 제사를 지내봤다.  
![image](https://user-images.githubusercontent.com/27724108/189506484-be34639f-6887-4bda-bae7-3810eba179e5.png)  

DPDK? 이건 kernel 에서 패킷 다이렉트로 가져올 때 쓰는 거잖아...? 이거 뭐임? Broadcom FlexSparc???  
![image](https://user-images.githubusercontent.com/27724108/189506510-22b4fc3c-4eb5-4ea4-b35b-4644818325b1.png)  

구글신이 답을 주긴 커녕 그냥 오히려 신탁이라는 엿을 줬다.  
하....  

![image](https://user-images.githubusercontent.com/27724108/189506578-3008840d-433a-4c7c-ba66-728402138059.png)  

## Broadcom FileSystem을 찾아서. (Part 2. 모든 것을 Micro$oft 에 맡긴 "김명자 낙지마당")
그래도 희망을 저버리고 싶진 않다. 적어도 난 [ghidra](https://github.com/NationalSecurityAgency/ghidra) 에서 MIPS32 어셈블리 일일이 읽으면서 하고 싶진 않기 때문이다....  

김명자 낙지마당에서는 GitHub Code Search 라고 코드를 왕창 뒤져볼 수 있는 무언가를 제공해준다. 이걸로 찾아보자.  
![image](https://user-images.githubusercontent.com/27724108/189506644-88d292c1-c305-455f-90a5-08a0982cf193.png)  
  
즈에발 이라면서 bcmfs 와 BFS 키워드로 "모든것을 Micro$oft에 맡긴 김명자 낙지마당" 에게 물어봤다.  
<img src="https://user-images.githubusercontent.com/27724108/189506673-fc850f58-4583-44c3-9268-f2393c98944d.png" width="300"/>

어? WRT? 이거 공유기 펌웨어 그런거 아님?  
![image](https://user-images.githubusercontent.com/27724108/189506741-376bfb1e-eb1c-4249-82e5-ec9a8df4e067.png)

생각해 보니 임베디드 해킹이면 OpenWRT 레포를 볼 생각을 안해봤다는 것도 나 자신이 🅱️🆎🅾️ 라는 걸 증명하는 것 같았다.
