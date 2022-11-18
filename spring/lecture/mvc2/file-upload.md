# 파일 업로드
## 소개

### **HTML 폼 전송 방식**

- application/x-www-form-urlencoded
- multipart/form-data

### **application/x-www-form-urlencoded 방식**
![img.png](../../../image/image-post.png)

- application/x-www-form-urlencoded 방식은 HTML 폼 데이터를 서버로 전송하는 가장 기본적인 방법
- .Form 태그에 별도의 enctype 옵션이 없으면 Content-Type: application/x-www-form-urlencoded 추가
- 그리고 폼에 입력한 전송할 항목을 HTTP Body에 문자로

  username=kim&age=20와 같이 & 로 구분해서 전송한다.


### **파일 업로드 시 위 방식의 문제점**

- 파일을 업로드 하려면 파일은 문자가 아니라 바이너리 데이터를 전송해야 한다. 문자를 전송하는 이 방식으로 파일을 전송하기는 어렵다
- 폼을 전송할 때 파일만 전송하는 것이 아니라는 점이다.
    - 이름과 나이, 첨부파일을 함께 전송해야 하는 경우
    - 문자와 바이너리를 동시에 전송해야 하는 상황

### **multipart/form-data 방식**
![img_1.png](../../../image/imgage-multipart.png)

- 이 방식을 사용하려면 Form 태그에 별도의 enctype="multipart/form-data"를 지정
- multipart/form-data 방식은 다른 종류의 여러 파일과 폼의 내용 함께 전송할 수 있다
- 폼의 입력 결과로 생성된 HTTP 메시지를 보면 각각의 전송 항목이 구분이 되어있다 (각각의 항목을 구분해서, 한번에 전송하는 것)
- 폼의 일반 데이터는 각 항목별로 문자가 전송되고, 파일의 경우 파일 이름과 Content-Type이 추가되고 바이너리 데이터가 전송된다.

### **Part**

multipart/form-data는 application/x-www-form-urlencoded와 비교해서 매우 복잡하고 각각의 부분(Part) 로 나누어져 있다. 그렇다면 이렇게 복잡한 HTTP 메시지를 서버에서 어떻게 사용할 수 있을까?

`Collection<Part> parts = request.getParts();`

- `HttpServletRequest`에서 part를 꺼낼 수 있다.
- `part.getSubmittedFileName()` : 클라이언트가 전달한 파일명
    - 예) 이미지 파일 이름 (이미지.png의 ‘이미지’)
- `part.getInputStream()`: Part의 전송 데이터를 읽을 수 있다.
- `part.write(...)`: Part를 통해 전송된 데이터를 저장할 수 있다.
    - 매개 변수로 저장 경로를 설정할 수 있다.

### **멀티파트 사용 옵션**

**업로드 사이즈 제한**

- `servlet.multipart.max-file-size=1MB`
    - 파일 하나의 최대 사이즈, 기본 1MB
- `servlet.multipart.max-request-size=10MB`
    - 멀티파트 요청 하나에 여러 파일을 업로드 할 수 있는데, 그 전체 합이다. 기본 10MB

**`spring.servlet.multipart.enabled` 끄기**

- `servlet.multipart.enabled=false` (기본 true)
- `servlet.multipart.enabled` 옵션을 끄면 서블릿 컨테이너는 멀티파트와 관련된 처리를 하지 않는다.
- 그래서 결과 로그 `request.getParameter("itemName"), request.getParts()` 의 결과가 비어있다.

## 스프링과 파일 업로드

스프링은 `MultipartFile`이라는 인터페이스로 멀티 파트 파일을 매우 편리하게 지원한다.

### **SpringUploadController**

```java
@Controller
@RequestMapping("/spring")
public class SpringUploadController {

   @Value("${file.dir}")
   private String fileDir;

   @GetMapping("/upload")
   public String newFile() {
      return "upload-form";
   }

   @PostMapping("/upload")
   public String saveFile(@RequestParam String itemName, @RequestParam MultipartFile file, HttpServletRequest request) throws IOException {

      if (!file.isEmpty()) {
         String fullPath = fileDir + file.getOriginalFilename();
         file.transferTo(new File(fullPath));
      }

      return "upload-form";
   }
}
```

- `@RequestParam MultipartFile file`
- 업로드하는 HTML Form의 name에 맞추어 `@RequestParam`을 적용
- 추가로`@ModelAttribute`에서도 MultipartFile을 동일하게 사용할 수 있다.
- **MultipartFile 주요 메서드**
    - `getOriginalFilename()` : 업로드 파일 명
    - `transferTo(...)` : 파일 저장

### JSON 타입과 MultipartFile 한 번에 받기

```java
@PostMapping(value = "/api/v1/character", consumes = {MediaType.APPLICATION_JSON_VALUE, MediaType.MULTIPART_FORM_DATA_VALUE})
public void saveCharacter(@RequestPart CharacterCreateRequest request,
                          @RequestPart MultipartFile imgFile) {
    // ...
}
```

- 특이점으로는 API 에서 **consume**할 **MediaType**을 지정해줘야 한다는 점이다.
    - 안하면 415 Unsupported MediaType ERROR

## 파일 저장 예시

```java
@Data
public class UploadFile {

		private String uploadFileName; // 고객이 업로드한 파일명
		private String storeFileName; // 서버 내부에서 관리하는 파일 명

		public UploadFile(String uploadFileName, String storeFileName) {
				this.uploadFileName = uploadFileName;
				this.storeFileName = storeFileName;
		}
}
```

```java
@Component
public class FileStore {

		@Value("${file.dir}")
		private String fileDir;

		public String getFullPath(String filename) {
				return fileDir + filename;
		}

		public List<UploadFile> storeFiles(List<MultipartFile> multipartFiles) throws IOException {
			List<UploadFile> storeFileResult = new ArrayList<>();
			for (MultipartFile multipartFile : imageFiles) {
			    if (!imageFile.isEmpty()) {
			        storeFileResult.add(storeFile(multipartFile));
			    }
			}
			return storeFileResult;
		}

		public UploadFile storeFile(MultipartFile multipartFile) throws IOException {
			if (multipartFile.isEmpty()) {
			    return null;
			}
			String originalFilename = multipartFile.getOriginalFilename();
			String storeFileName = createStoreFileName(originalFilename);
			multipartFile.transferTo(new File(getFullPath(storeFileName)));
			return new UploadFile(originalFilename, storeFileName);
		} 

		private String createStoreFileName(String originalFilename) {
			String ext = extractExt(originalFilename);
			String uuid = UUID.randomUUID().toString();
			return uuid + "." + ext;
		}

		private String extractExt(String originalFilename) {
			int pos = originalFilename.lastIndexOf(".");
			return originalFilename.substring(pos + 1);
		}
}
```

- 멀티 파트 파일을 서버에 저장하는 역할을 담당한다.
    - `createStoreFileName()` : 서버 내부에서 관리하는 파일명은 유일한 이름을 생성하는 UUID 를 사용해서 충돌하지 않도록 한다.
    - `extractExt()` : 확장자를 별도로 추출해서 서버 내부에서 관리하는 파일명에도 붙여준다. 예를 들어서 고객이 a.png 라는 이름으로 업로드 하면 51041c62-86e4-4274-801d614a7d994edb.png 와 같이 저장한다

## 파일 조회

```java
@ResponseBody
@GetMapping("/images/{filename}")
public Resource downloadImage(@PathVariable String filename) throws MalformedURLException {
		return new UrlResource("file:" + fileStore.getFullPath(filename));
}
```

`<img>` 태그로 이미지를 조회할 때 사용한다. `UrlResource`로 이미지 파일을 읽어서 `@ResponseBody`로 이미지 바이너리를 반환한다.
