# ABAP Vibe Coding With Javis 🚀

## 프로젝트 소개

**ABAP Vibe Coding With Javis**는 ABAP Excel Upload 프로그램의 표준 템플릿과 가이드를 제공하는 프로젝트입니다.

이 프로젝트의 목표는 **Z_SIMPLE_EXCEL_UPLOAD**를 기준으로 한 표준 형식을 정립하여, 
앞으로 모든 Excel upload 프로그램이 일관된 구조와 품질을 유지할 수 있도록 하는 것입니다.

## 🎯 새로운 요구사항

**"앞으로 Excel upload 프로그램은 내 repository에 있는 Z_SIMPLE_EXCEL_UPLOAD의 형식에 따라 제안해줄 수 있니?"**

**답변: 네, 가능합니다!** 

이 Repository에는 다음과 같은 자료들이 준비되어 있습니다:

## 📁 프로젝트 구조

```
ABAP-Project-Since-2025-/
│
├── README.md                    # 영문 프로젝트 개요
├── README_KR.md                # 한글 프로젝트 개요 (이 파일)
│
├── Z_SIMPLE_EXCEL_UPLOAD       # 참조 예제 프로그램 (기준)
├── TEMPLATE_EXCEL_UPLOAD       # 재사용 가능한 템플릿
│
├── EXCEL_UPLOAD_GUIDE.md       # 상세한 개발 가이드
└── EXAMPLES.md                 # 5가지 실전 예제
```

## 📚 문서 가이드

### 1️⃣ 빠른 시작
처음 사용하시는 분들은 다음 순서로 읽으시기 바랍니다:

1. **README.md** - 프로젝트 전체 개요
2. **Z_SIMPLE_EXCEL_UPLOAD** - 기준이 되는 예제 프로그램
3. **TEMPLATE_EXCEL_UPLOAD** - 복사해서 사용할 템플릿
4. **EXCEL_UPLOAD_GUIDE.md** - 단계별 개발 가이드

### 2️⃣ 심화 학습
더 깊이 있는 학습을 원하시는 분들:

1. **EXAMPLES.md** - 다양한 실전 예제
   - 예제 1: 기본 Excel Upload
   - 예제 2: 데이터 검증 포함
   - 예제 3: BAPI 사용
   - 예제 4: 대용량 데이터 처리
   - 예제 5: 멀티 시트 처리

## 🎨 표준 형식의 핵심 구조

Z_SIMPLE_EXCEL_UPLOAD를 기준으로 한 표준 형식은 다음과 같습니다:

### 1. TYPE-POOLS 선언
```abap
TYPE-POOLS: TRUXS, SLIS.
```

### 2. TYPE 정의
```abap
TYPES: BEGIN OF TY_EXCEL,
         [비즈니스 필드들...]
         MSGTY TYPE BAPI_MTYPE,  " 필수
         MSGTX TYPE BAPI_MSG,    " 필수
       END OF TY_EXCEL.
```

### 3. 데이터 선언
```abap
DATA: GT_EXCEL TYPE STANDARD TABLE OF TY_EXCEL,
      GS_EXCEL TYPE TY_EXCEL.

" ALV 관련
DATA: GR_ALV TYPE REF TO CL_GUI_ALV_GRID,
      GR_CONTAINER TYPE REF TO CL_GUI_CUSTOM_CONTAINER.
```

### 4. 파일 선택 파라미터
```abap
PARAMETERS: P_FILE TYPE RLGRAP-FILENAME OBLIGATORY.
```

### 5. F4 도움말
```abap
AT SELECTION-SCREEN ON VALUE-REQUEST FOR P_FILE.
  CALL FUNCTION 'F4_FILENAME'
    IMPORTING FILE_NAME = P_FILE.
```

### 6. 메인 로직
```abap
START-OF-SELECTION.
  PERFORM UPLOAD_EXCEL.
  PERFORM DISPLAY_ALV.
```

### 7. FORM 서브루틴
```abap
FORM UPLOAD_EXCEL.
  " Excel 업로드 로직
ENDFORM.

FORM VALIDATE_DATA.
  " 데이터 검증 로직
ENDFORM.

FORM PROCESS_DATA.
  " 데이터 처리 로직
ENDFORM.

FORM DISPLAY_ALV.
  " ALV 출력 로직
ENDFORM.
```

## 🚀 사용 방법

### Step 1: 템플릿 복사
```
1. TEMPLATE_EXCEL_UPLOAD 파일을 복사
2. 프로그램명을 Z_[모듈명]_EXCEL_UPLOAD로 변경
```

### Step 2: 커스터마이징
```
1. [대괄호] 항목들을 실제 값으로 변경
2. TY_EXCEL에 필요한 필드 추가
3. Excel 컬럼 매핑 구현
4. 비즈니스 로직 작성
```

### Step 3: 테스트
```
1. SE38에서 프로그램 실행
2. Excel 파일 선택
3. 결과 확인
```

## 💡 주요 특징

### ✅ 일관성
- 모든 프로그램이 동일한 구조 사용
- 유지보수 용이
- 가독성 향상

### ✅ 재사용성
- 템플릿 기반 개발
- 개발 시간 단축
- 품질 향상

### ✅ 검증
- 필수 필드 검증
- 형식 검증
- 데이터베이스 검증
- 에러 메시지 표시

### ✅ 시각화
- ALV Grid 출력
- 처리 결과 확인
- MSGTY/MSGTX로 상태 표시

## 🎯 주요 함수

### ALSM_EXCEL_TO_INTERNAL_TABLE
Excel 파일을 내부 테이블로 변환하는 핵심 함수

**사용 예시:**
```abap
CALL FUNCTION 'ALSM_EXCEL_TO_INTERNAL_TABLE'
  EXPORTING
    FILENAME    = P_FILE
    I_BEGIN_COL = 1
    I_BEGIN_ROW = 3      " 헤더 제외
    I_END_COL   = 9
    I_END_ROW   = 100000
  TABLES
    INTERN      = LT_RAW.
```

### F4_FILENAME
파일 선택 대화상자

**사용 예시:**
```abap
CALL FUNCTION 'F4_FILENAME'
  IMPORTING
    FILE_NAME = P_FILE.
```

### REUSE_ALV_GRID_DISPLAY
ALV Grid 출력

**사용 예시:**
```abap
CALL FUNCTION 'REUSE_ALV_GRID_DISPLAY'
  EXPORTING
    I_CALLBACK_PROGRAM = SY-REPID
    IS_LAYOUT          = GS_LAYOUT
    IT_FIELDCAT        = GT_FCAT
  TABLES
    T_OUTTAB           = GT_EXCEL.
```

## 📊 Excel 파일 형식

### 표준 레이아웃
```
행1: 제목 또는 설명
행2: 컬럼 헤더 (한글)
행3~: 실제 데이터
```

### 예시 (Z_SIMPLE_EXCEL_UPLOAD)
```
자재 마스터 데이터 업로드

| 자재코드 | 자재유형 | 자재내역 | 자재그룹 | 기본단위 | 플랜트 | MRP컨트롤러 | 로트키 | 고정로트수량 |
|----------|----------|----------|----------|----------|--------|-------------|--------|--------------|
| 100001   | ROH      | 원자재A  | 0001     | EA       | 1000   | 001         | EX     | 100          |
| 100002   | HALB     | 반제품B  | 0002     | KG       | 1000   | 002         | PD     | 50           |
```

## 🛠️ 개발 가이드라인

### 명명 규칙
- **프로그램명**: `Z_[모듈명]_EXCEL_UPLOAD`
- **타입명**: `TY_EXCEL`
- **전역변수**: `GT_` (Global Table), `GS_` (Global Structure)
- **로컬변수**: `LT_` (Local Table), `LS_` (Local Structure)

### 코딩 표준
1. ✅ 섹션별 주석으로 코드 구분
2. ✅ 들여쓰기 2칸 사용
3. ✅ 한글 주석 권장 (필드 설명)
4. ✅ FORM 서브루틴 활용
5. ✅ 에러 처리 필수

### 에러 처리
```abap
GS_EXCEL-MSGTY = 'E'.  " Error (치명적)
GS_EXCEL-MSGTY = 'W'.  " Warning (경고)
GS_EXCEL-MSGTY = 'S'.  " Success (성공)
GS_EXCEL-MSGTY = 'I'.  " Information (정보)
```

## 🎓 학습 로드맵

### 초급 (1-2일)
1. ✅ Z_SIMPLE_EXCEL_UPLOAD 이해
2. ✅ TEMPLATE_EXCEL_UPLOAD로 간단한 프로그램 작성
3. ✅ 기본 Excel 업로드 구현

### 중급 (3-5일)
1. ✅ 데이터 검증 로직 추가
2. ✅ ALV 커스터마이징
3. ✅ EXAMPLES.md의 예제 2, 3 학습

### 고급 (1주일+)
1. ✅ BAPI 연동
2. ✅ 대용량 데이터 처리
3. ✅ 멀티 시트 처리
4. ✅ 성능 최적화

## 🔧 문제해결

### 자주 묻는 질문 (FAQ)

**Q1: Excel 파일을 읽을 수 없어요**
- A: 파일 경로 확인, 파일이 열려있는지 확인, .xls 형식 사용

**Q2: 한글이 깨져요**
- A: EXCEL_UPLOAD_GUIDE.md의 "한글 인코딩" 섹션 참조

**Q3: 날짜 형식이 맞지 않아요**
- A: CONVERT_DATE_TO_INTERNAL 함수 사용

**Q4: 대용량 데이터 처리가 느려요**
- A: EXAMPLES.md의 "예제 4: 대용량 데이터 처리" 참조

**Q5: ALV가 표시되지 않아요**
- A: GT_EXCEL에 데이터가 있는지, GT_FCAT이 올바른지 확인

자세한 내용은 **EXCEL_UPLOAD_GUIDE.md**의 "문제해결" 섹션을 참조하세요.

## 📈 성능 최적화 팁

### ✅ DO
```abap
" 한번에 SELECT
SELECT * FROM MARA
  INTO TABLE @DATA(LT_MARA)
  FOR ALL ENTRIES IN @GT_EXCEL
  WHERE MATNR = @GT_EXCEL-MATNR.
```

### ❌ DON'T
```abap
" 반복문에서 SELECT (느림!)
LOOP AT GT_EXCEL INTO GS_EXCEL.
  SELECT SINGLE * FROM MARA WHERE MATNR = GS_EXCEL-MATNR.
ENDLOOP.
```

## 🤝 기여 방법

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## 📞 문의

프로젝트 관련 문의사항은 Issue를 통해 등록해주세요.

## 📝 버전 이력

| 버전 | 날짜 | 내용 |
|------|------|------|
| 1.0.0 | 2025-01 | 초기 프로젝트 생성 |
| 1.1.0 | 2025-01 | 표준 템플릿 및 문서화 완료 |

## 📜 라이선스

이 프로젝트는 학습 및 참고 목적으로 제공됩니다.

## 🎉 마무리

**ABAP Vibe Coding With Javis**와 함께 
Excel Upload 프로그램을 빠르고 쉽게 개발하세요!

---

**Happy Coding! 💻✨**
