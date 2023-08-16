# 아이템 9. try-finally 보다는 try-with-resources를 사용하라

- 자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야 하는 자원이 많다.
    - ex) `InputStream`, `OutputStream`, `java.sql.Connection`

## try-finally

- 전통적으로 자원을 닫을 때 `try-finally`를 많이 사용했다.

    ```java
    static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }
    ```

    - 위 방식엔 단점이 있는데 하드웨어 에러로 인해 `try` 내부와 `finally` 내부에서 모두 예외가 발생하면 두 번째 발생한 예외가 첫 번째 발생한 예외를 집어삼켜 버린다.
    - 스택 추적 내역에 첫 번째 예외가 기록되지 않기 때문인데 이를 기록하도록 코드를 수정할 순 있지만 코드가 지저분해진다.
- 또 하나의 단점으로 자원이 추가될 때마다 코드가 지저분해진다.

    ```java
    static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src)
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new Byte[BUFFER_SIZE];
                int n;
                while ((n == in.read(buf)) >= 0) {
                    out.write(buf, 0, n);
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    }
    ```


## try-with-resources

- 위 문제들을 해결하기 위해 자바 7 이후에 `try-with-resources`가 추가되었다.
- 이 구조를 사용하려면 해당 자원이 `AutoCloseable` 인터페이스를 구현해야 한다.
    - void 타입의 `close()` 메서드만 구현하면 된다.
    - 자바 라이브러리와 서드파티 라이브러리들은 이미 `AutoCloseable`을 구현한 상태다.
- 복수의 자원을 처리하는 `try-with-resources` 코드

    ```java
    static void copy(String src, String dst) throws IOException {
        try (InputStream in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dist)) {
            byte[] buf = new Byte[BUFFER_SIZE];
            int n;
            while ((n == in.read(buf)) >= 0) {
                out.write(buf, 0, n);
            }
        }
    }
    ```

    - 짧고 읽기 수월할 뿐 아니라 문제를 진단하기도 훨씬 좋다.
- 로직과 자원을 닫는 부분이 함께 예외를 발생시키는 경우에도 예외가 다 기록되기 때문에 디버깅이 쉬워진다.
