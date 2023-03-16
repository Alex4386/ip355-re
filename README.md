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

사실 BFS 를 직접 찾을 필요는 없었다. 뭐 굳이 삽질을 보고 싶다면, [BCMFS_SAPJIL.md](BCMFS_SAPJIL.md) 참고.  


## 발상의 전환
하지만 나는 다른 방법으로 찾아내기로 결심 했다.  
인터넷에 잘 찾아보면, 고등학교 교무실에서 굉장히 흔하게 볼 수 있던 IP255S 의 펌웨어가 올라와 있다. IP355 는 잘 안보인다... :(  
  
이걸 역 이용하자.  

우선 우리는 BFS 시그니쳐를 통해 BFS 의 시작이 `0x40000` 이라는 걸 알 수 있다.  
![021DE08F-3515-41AC-B63F-34F4A7360E9B](https://user-images.githubusercontent.com/27724108/225640929-5057dd44-363f-491f-8f10-7be9b6a6b923.jpeg)  
  
그렇다면, 여기서 파일 이름 struct (UART를 통해 단순한 파일 시스템이 구현되어있는걸 확인 할 수 있었음 + IP355 웹 콘솔) 를 대충 추정해 보자.  
  
우선 그렇게 하려면 Filesystem inode header 의 구간이 어디까지인지를 알아야 한다.  
이걸 상대적으로 용이하게 하기 위해, IP255S 의 펌웨어를 바탕으로 IP355의 펌웨어 파일이 어디서 시작하는 지를 대충 찍어볼 수 있다!!!  
![3ED0D997-E34E-488A-8DB6-6DE01358CB6A](https://user-images.githubusercontent.com/27724108/225641147-b71bd03f-6449-44be-8b27-e79c98d5a43f.jpeg)  

이제 이걸 보니 조금 감이 잡히는 것 같다.  
일단, `BFS` header 같아 보이는 첫 `0x10` 부분까지는 건너 뛰자.  
(아마 이 BFS 오브젝트의 사이즈를 나타내는 것 같다. 이 BFS 오브젝트 길이는 `0x178100` 인 듯)  

다음은 `0x20` 부분 이다. 숫자를 세어보면 알겠지만 파일 이름 `ip355_20121112_v110008_s.sto`, 이거 NULL 바이트를 빼고 바이트 수 세어 보면 28바이트, 즉 `0x1c` 다. 파일 이름 길이 바이트 수 인걸로 추정된다.  

그 뒤는 아마 `0x04` 개의 qword 를 지나야 나온다는 거 같긴 한데... 맞는진 모르겠다.  

그 뒤에 나오는 바이트는 아무리 대충 봐도 파일 이름일꺼고...  
그 뒤를 보면 나오는 `0x1780a2`, 이 쯤되면 눈치가 빠른 사람들은 이미 눈치를 깠을 것이다. 왜인지 용량 일것 같다.  

hexdump 로 거기까지 한번 가보자.
![29B69C8F-6230-41E3-8F3D-5BAED140635D](https://user-images.githubusercontent.com/27724108/225650760-03c5cd02-2d55-447c-8562-c3aa1cb46b9d.jpeg)

헤더 오프셋이 `0x50` 이므로 그걸 적용해서 가져오면 정확히 맞아 떨어진다. 심지어 해당 파일 이후로는 `0xff` 가 도배되는 것으로 보아 NOR Flash의 기록되지 않은 부분이 나오는 걸로 보아 여기가 끝이 맞다!  

그럼 이제 대충 위치를 잡아 node 를 통해 파일을 적출해 낼 수 있다!
![F3BE1F1E-7DE0-4498-8BFB-3CF56952C990](https://user-images.githubusercontent.com/27724108/225654800-c80ffecc-74f6-41b3-82dd-ca020a81649b.jpeg)

그리고 더욱 재밌는 사실을 알아낼 수 있다. 보면 알 수 있듯, 문자열이 대놓고 나와있다.  
이 뜻은, 이 파일이 암호화나 압축이 되어있지 않거나, 이 부분에 대해서는 되어있지 않다는 걸 의미한다.  
  
![81357EAA-A763-457F-94A4-1E9937E3F4FC](https://user-images.githubusercontent.com/27724108/225641431-9708e018-b336-4d5d-8447-12bdcaed63f2.jpeg)  

이제 플래시 덤프에서 `.sto` 파일을 적출해 낼 수 있다.

