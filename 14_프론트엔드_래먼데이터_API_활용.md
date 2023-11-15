# 프론트엔드의 래먼데이터 API 활용

## backend 서버 start
- ### lahmansbaseballdb.sqlite가 있는 폴더 열기 (VsCode-File-Open Folder)

- ### mlb_crud.py 생성
    ```python
    from sqlalchemy import create_engine, text


    SQLALCHEMY_DATABASE_URL = "sqlite:///./lahmansbaseballdb.sqlite"

    engine = create_engine(
        SQLALCHEMY_DATABASE_URL
    )


    def read_player_batting_data(playerID: str):
        with engine.connect() as conn:
            rows = conn.execute(
                text("select * from batting where playerID = :playerID"),
                {'playerID': playerID}
            )

        columns = rows.keys()

        data = []
        for row in rows:
            data_dict = {column: row[idx] for idx, column in enumerate(columns)}
            data.append(data_dict)

        return data


    def read_player_pitching_data(playerID: str, yearID: str):
        with engine.connect() as conn:
            rows = conn.execute(text("select * from pitching where playerID = :playerID AND yearID = :yearID"), {'playerID': playerID, 'yearID': yearID})
            
        columns = rows.keys()
        
        data = []
        for row in rows:
            data_dict = {column: row[idx] for idx, column in enumerate(columns)}
            data.append(data_dict)
        
        return data


    def read_player_info(playerID: str):
        with engine.connect() as conn:
            rows = conn.execute(text("select * from people where playerID = :playerID"), {'playerID': playerID})
            
        columns = rows.keys()
        
        data = []
        for row in rows:
            data_dict = {column: row[idx] for idx, column in enumerate(columns)}
            data.append(data_dict)
        
        return data
    ```

- ### mlb_api.py 생성
    ```python
    from fastapi import FastAPI, Body
    from fastapi.middleware.cors import CORSMiddleware # 14_프론트엔드_래먼데이터_API_활용 추가

    import mlb_crud


    app = FastAPI(title="레먼데이터베이스 API")
    
    # 14_프론트엔드_래먼데이터_API_활용 추가
    app.add_middleware(
        CORSMiddleware,
        allow_origins=["*"],
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"]
    )

    @app.get("/players/{playerID}")  # 경로 매개변수 활용 예시
    def player_info_route(playerID: str):
        return crud.read_player_info(playerID)


    @app.get("/batting")  # 쿼리 매개변수 활용 예시
    def batting_route(playerID: str):
        return crud.read_player_batting_data(playerID)


    @app.post("/pitching")  # Request Body 활용 예시
    def pitching_route(
        playerID: str = Body(...),
        season: int = Body(...)
    ):
        return crud.read_player_pitching_data(playerID, season)
    ```

- ### 실행

  - #### 윈도우
    ```bash
    call .venv/Scripts/activate
    uvicorn mlb_api:app --host 0.0.0.0 --port 8999 --reload
    ```
  - #### MacOS 
    ```bash
    source .venv/bin/activate
    uvicorn mlb_api:app --host 0.0.0.0 --port 8999 --reload
    ```
---

## frontend 류현진 데이터 목록 조회
GET : https://kusf-api.run.goorm.site/pitching/seasons?playerID=ryuhy01&start_year=2010&end_year=2020



### vscode 실행

#### front.html 작성
``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>

    <script src="https://code.jquery.com/jquery-3.7.0.min.js" integrity="sha256-2Pmvv0kuTBOenSvLm6bvfBSSHrUJ+3A7x6P5Ebd07/g=" crossorigin="anonymous"></script>
</head>

<script>
    //모든 html이 로딩 되었을 때, 실행
    $(function() {
        getPlayerInfo();
    });

    function getPlayerInfo() {
        $.ajax ({
            url	: "http://localhost:8999/players/ryuhy01",                                // 요청이 전송될 URL 주소
            type	: "GET",                            // http 요청 방식 (default: ‘GET’)
            //data  : {key : value},                    // 요청 시 포함되어질 데이터
            success : function(data, status, xhr) {     // 정상적으로 응답 받았을 경우에는 success 콜백이 호출되게 됩니다.
                console.log(data);
                $("#playerInfo").text(JSON.stringify(data));
                /*
                JSON.stringify : javascript 객체를 JSON 문자열로 변환
                JSON.parse : JSON 문자열을 javascript 객체로 변환
                */
            },
            error	: function(xhr, status, error) {    // 서버에서 error가 생겼을 때 호출됩니다.
                console.log(xhr);                      
            },
        });
    }
</script>

<body>
    <div id="playerInfo">
    </div>
</body>
</html>
```
### 결과
![14_01.png](./images/14_01.png)