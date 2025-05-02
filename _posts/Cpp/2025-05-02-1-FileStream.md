# C 스타일 파일 입출력

File Stream을 만들어서 FILE 타입의 변수에 가지고 있게 해준다.
File Stream은 파일을 만들거나 읽어올 때 해당 파일과 연결된 변수이다.

절대경로: 보통 탐색기의 폴더 경로. 드라이브부터의 전체 경로.
상대경로: 현재 폴더를 기준으로 목적지 경로까지의 상대적인 경로.

|모드 문자열|의미|파일이 없을 때|파일이 있을 때|읽기|쓰기|추가|
|:-----:|:---:|:-------:|:-------:|:---:|:---:|:---:|
r|읽기|오류 발생|파일열기|O|X|X|
w|쓰기|파일 생성|덮어쓰기|X|O|X|
a|추가|파일 생성|내용 추가|X|O|O|
r+|읽기/쓰기|오류발생|파일 열기|O|O|X|
w+|읽기/쓰기|새로 생성|덮어쓰기|O|O|X|
a+|읽기/추가|새로생성|열기, 추가|O|O|O|
rt|텍스트읽기|오류발생|열기|O|x|x|



## fopen_s, fclose, fputs, fputc, fgets, fgetc

```cpp
#include <iostream>

int main()
{
    FILE* fileStream = nullptr;

    // 1번 인자에 FileStream을 만들어준다.
    // 2번 인자는 파일의 경로가 들어가는데, 보통 상대경로를 사용하며,
    // 상대경로의 경우 현재 프로젝트 파일이 있는 경로를 기준으로 지정한다.
    // 3번 인자는 파일을 만들거나 읽을 때, 어떤 파일을 만들거나 읽을지 옵션을 지정하는 기능이다.
    // 첫번째 : r, w, a
    // 두번째 : t (텍스트파일), b (바이너리파일), + 

    // 파일 쓰기

    // 텍스트파일 생성
    // 함수의 인자로 이중포인터를 설정하여,
    // 함수 바깥의 변수 주소를 받아와서 그 변수에 동적할당 하는 원리와 같다.
    fopen_s(&fileStream, "Test.txt", "wt");

    // 파일 스트림이 있을 경우(nullptr이 아니라면), 파일을 만들 수 있다는 의미이다.
    if (fileStream)
    {
        // fputs : 파일에 텍스트를 저장해주는 함수
        fputs("0xFEFF", fileStream);

        fputs("File Stream !", fileStream);

        fclose(fileStream);
    }



    // 파일 읽기

    fopen_s(&fileStream, "Test.txt", "rt");

    if (fileStream)
    {
        char firstText = fgetc(fileStream);

        printf("firstText : %c", firstText);

        char text[256] = {};

        // 개행문자가 있는 곳 까지 읽어온다.
        // 만약 개행문자 없이 파일의 끝에 도달했다면, 거기까지만 읽어오게 된다.
        fgets(text, 256, fileStream);

        printf("text : %s", text);

        fclose(fileStream);
    }

    return 0;
}
```



## 바이너리 파일 쓰고 읽기 (배열, 구조체)

`fwrite(&number, sizeof(int), 1, fileStream);`
1번 인자의 주소로부터 <mark style='background-color: #fff5b1'>2번 인자의 크기 * 3번 인자의 갯수 </mark>만큼의 바이트를  
파일에 복사한다.  

`fread(&number, sizeof(int), 1, fileStream);`



```cpp
#include <iostream>

enum class EPlayerJob : unsigned char
{
	None,
	Knight,
	Archer,
	Magicion
};

struct FPlayerInfo
{
	char Name[32] = {};
	EPlayerJob Job = EPlayerJob::Knight;
	int Attack = 0;
	int Defense = 0;
	int Hp = 0;
	int HpMax = 0;
	int Mp = 0;
	int MpMax = 0;
};

int main()
{
	/*
	바이너리파일: 특정 메모리의 데이터를 그대로 파일에 옮겨서 저장할 때 사용한다.
	*/

	FILE* fileStream = nullptr;

	fopen_s(&fileStream, "TestBinary.tbf", "wb");

	if (fileStream)
	{
		int number = 100;

		// 1번 인자의 주소로부터 2번 인자의 크기 * 3번 인자의 갯수 만큼의 바이트를
		// 파일에 복사한다.
		fwrite(&number, sizeof(int), 1, fileStream);


		// 배열 저장
		int array[10] = {};

		for (int i = 0; i < 10; i++)
		{
			array[i] = i + 1;
		}

		fwrite(array, sizeof(int), 10, fileStream);

		
		// 구조체 저장
		FPlayerInfo player;
		strcpy_s(player.Name, "susu");
		player.Attack = 100;
		player.Defense = 50;
		player.Hp = 500;
		player.HpMax = 500;
		player.Mp = 100;
		player.MpMax = 300;

		fwrite(&player, sizeof(FPlayerInfo), 1, fileStream);

		fclose(fileStream);
	}

	fopen_s(&fileStream, "TestBinary.tbf", "rb");

	if (fileStream)
	{
		int number = 0;

		fread(&number, sizeof(int), 1, fileStream);

		printf("number : %d\n", number);

		// 구조체 읽어오기
		FPlayerInfo player;

		fread(&player, sizeof(FPlayerInfo), 1, fileStream);
		

		// 배열 읽어오기
		int array[10] = {};

		fread(array, sizeof(int), 10, fileStream);

		for (int i = 0; i < 10; i++)
		{
			printf("array [%d] : %d\n", i, array[i]);
		}

		fclose(fileStream);
	}

	return 0;
}
```



# C++ 스타일 파일 입출력

## ofstream(쓰기), ifstream(읽기), 

```cpp
#include <iostream>
#include <fstream>                      // fstsream 을 추가해야 사용할 수 있음.

int main()
{
    // 2번 인자에 std::ios::binary 을 넣어주면 바이너리 파일이 된다.
    std::ofstream writeFile("Test2.txt", std::ios::binary);

    // 파일을 만들 수 있는 상태인지 체크한다.
    if (writeFile.is_open() == false)
        return 0;

    writeFile << "텍스트 추가\n";
    writeFile << "텍스트 한 줄 더 추가\n";

    writeFile.close();



    // 읽어오기
    std::ifstream readFile("Test2.txt");

    if (readFile.is_open() == false)
        return 0;

    char text[256] = {};

    // 개행문자 전 까지 한 줄을 읽어오고, 개행문자는 미포함.
    readFile.getline(text, 256);

    // 문자만 읽어오고, 스페이스 바, 개행문자 등의 전 까지만 읽어온다.
    //readFile >> text;

    printf("text : %s", text);				// 텍스트 추가 출력

    readFile.getline(text, 256);
    printf("text : %s", text);				// 텍스트 한 줄 더 추가 출력

    return 0;
}
```



## 바이너리 파일에 구조체, 배열 읽고 쓰기

```cpp
#include <iostream>
#include <fstream>

int main()
{
	std::ofstream writeFile("TestBinary2.tbf", std::ios::binary);

	if (writeFile.is_open() == false)
		return 0;

	int number = 100;

	// const char*로 형변환을 해줘야 쓸 수 있다.
	writeFile.write((const char*)&number, sizeof(int));

	int array[10] = {};

	for (int i = 0; i < 10; i++)
	{
		array[i] = i + 1;
	}

	writeFile.write((const char*)array, sizeof(int) * 10);

	writeFile.close();


	// 읽어오기

	std::ifstream readFile("TestBinary2.tbf", std::ios::binary);

	if (readFile.is_open() == false)
		return 0;

	// 제대로 읽어오는지 보려고 초기화
	number = 0;
	for (int i = 0; i < 10; i++)
	{
		array[i] = 0;
	}

	readFile.read((char*)&number, sizeof(int));
    readFile.read((char*)array, sizeof(int) * 10);

    for (int i = 0; i < 10; i++)
    {
        printf("array[%d] : %d\n", i, array[i]);
    }

	readFile.close();
	

	return 0;
}
```