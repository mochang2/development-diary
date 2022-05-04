## 1. apt install

    sudo apt install openjdk-11-jdk  // jdk 다운받기. 11은 버전
    java -version
    javac -version  // 확인
    
__참고 사항: JVM, JRE, JDK의 차이__
JVM은 자바 프로그램이 어느 기기, 어느 운영체제 상에서도 실행될 수 있게 만들어 준다. 또한 자바 프로그램의 메모리를 효율적으로 관리(garbage collection 등) & 최적화해 준다. JRE는 JVM에 Library classes와 Java class loader가 추가된 것이고, JDK는 JRE에 Development Tools(자바 컴파일러 등)가 추가된 것이다.

## 2. 쉘에 환경변수 설정

    sudo gedit ~/.bashrc  // home의 bash 설정을 쓰는 파일
    
    // 아래 내용 추가
    # JAVA_HOME settings
    export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))  // JAVA_HOME 이라는 변수 설정
    export PATH=$PATH:$JAVA_HOME/bin  // 변수를 PATH에 추가
    
이후 

    source ~/.bashrc  // 쉘을 끄지 않고 이 쉘에 바로 변경 사항 적용

## 3. xx.java 파일을 만든 후 컴파일

    javac -d . xx.java  // 여기 디렉터리에 컴파일된 클래스 파일 생성. xx.class란 파일이 만들어 질 것임
    java -cp . xx  // cp는 classpath의 약자로, 컴파일된 파일을 실행시킴
    
