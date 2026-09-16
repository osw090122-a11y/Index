<!DOCTYPE html>

<html lang="ko">

<head>

<meta charset="UTF-8">

<meta name="viewport"
      content="width=device-width,
               initial-scale=1.0">

<title>Quiz Ranking</title>


<!-- PyScript -->

<link rel="stylesheet"
      href="https://pyscript.net/releases/2025.8.1/core.css">

<script type="module"
        src="https://pyscript.net/releases/2025.8.1/core.js">
</script>


<style>

* {
    box-sizing: border-box;
}


body {

    margin: 0;

    padding: 20px;

    background: #f3f4f6;

    font-family:
        Arial,
        sans-serif;
}


.container {

    width: 100%;

    max-width: 650px;

    margin: auto;
}


.title {

    text-align: center;

    margin-bottom: 25px;
}


.title h1 {

    margin-bottom: 5px;
}


.title p {

    color: #777;

    margin-top: 0;
}


.card {

    background: white;

    border-radius: 15px;

    padding: 20px;

    margin-bottom: 20px;

    box-shadow:
        0 4px 15px
        rgba(0,0,0,0.08);
}


/* 테스트 점수 등록 */

input {

    width: 100%;

    padding: 13px;

    margin-top: 7px;

    margin-bottom: 12px;

    border: 1px solid #ddd;

    border-radius: 8px;

    font-size: 16px;
}


button {

    width: 100%;

    padding: 13px;

    border: 0;

    border-radius: 8px;

    background: #222;

    color: white;

    font-size: 16px;

    cursor: pointer;
}


button:hover {

    opacity: 0.85;
}


/* 랭킹 */

.ranking-row {

    display: flex;

    align-items: center;

    padding: 15px 5px;

    border-bottom:
        1px solid #eee;
}


.ranking-row:last-child {

    border-bottom: none;
}


.rank {

    width: 60px;

    font-weight: bold;

    font-size: 18px;
}


.player {

    flex: 1;

    font-size: 17px;
}


.score {

    font-weight: bold;

    font-size: 17px;
}


.first {

    font-size: 22px;

    font-weight: bold;
}


.second {

    font-size: 20px;

    font-weight: bold;
}


.third {

    font-size: 20px;

    font-weight: bold;
}


#status {

    text-align: center;

    margin-top: 12px;

    color: #777;

    min-height: 20px;
}


#last-update {

    text-align: center;

    color: #999;

    font-size: 13px;

    margin-top: 15px;
}

</style>

</head>


<body>


<div class="container">


    <div class="title">

        <h1>🏆 Quiz Ranking</h1>

        <p>
            현재 참가자들의 최고 기록
        </p>

    </div>



    <!-- ==================================
         테스트용 점수 등록
         ================================== -->

    <div class="card">

        <h2>점수 등록</h2>

        <p>
            실제 퀴즈에서는 친구의 프로그램이
            이 API를 직접 호출하면 됩니다.
        </p>


        <input
            id="name"
            type="text"
            placeholder="이름">


        <input
            id="score"
            type="number"
            placeholder="점수">


        <button
            py-click="submit_score">

            점수 등록

        </button>


        <div id="status"></div>

    </div>



    <!-- ==================================
         랭킹
         ================================== -->

    <div class="card">

        <h2>🏆 현재 랭킹</h2>


        <div id="ranking">

            랭킹을 불러오는 중...

        </div>


        <div id="last-update"></div>

    </div>


</div>



<!-- ======================================
     PyScript
     ====================================== -->

<script type="py">

from pyscript import document
from js import fetch, JSON
import asyncio


# ==========================================
# 랭킹 불러오기
# ==========================================

async def load_ranking():

    try:

        response = await fetch(
            "/api/ranking"
        )

        data = await response.json()

        ranking_box = document.getElementById(
            "ranking"
        )

        ranking_box.innerHTML = ""


        if len(data) == 0:

            ranking_box.innerHTML = """
                <p style="text-align:center;">
                    아직 등록된 기록이 없습니다.
                </p>
            """

            return


        # ------------------------------
        # 랭킹 표시
        # ------------------------------

        for index, player in enumerate(data):

            rank = index + 1

            name = str(
                player["name"]
            )

            score = player["score"]


            row = document.createElement(
                "div"
            )

            row.className = "ranking-row"


            # 순위
            rank_box = document.createElement(
                "div"
            )

            rank_box.className = "rank"

            rank_box.innerText = (
                str(rank) + "위"
            )


            # 이름
            player_box = document.createElement(
                "div"
            )

            player_box.className = "player"

            player_box.innerText = name


            # 점수
            score_box = document.createElement(
                "div"
            )

            score_box.className = "score"

            score_box.innerText = (
                str(score) + "점"
            )


            row.appendChild(rank_box)

            row.appendChild(player_box)

            row.appendChild(score_box)

            ranking_box.appendChild(row)


        # 업데이트 시간

        update_box = document.getElementById(
            "last-update"
        )

        update_box.innerText = (
            "자동 갱신됨"
        )


    except Exception as e:

        ranking_box = document.getElementById(
            "ranking"
        )

        ranking_box.innerHTML = """
            <p style="text-align:center;">
                서버에 연결할 수 없습니다.
            </p>
        """


# ==========================================
# 점수 제출
# ==========================================

async def submit_score(event=None):

    name_box = document.getElementById(
        "name"
    )

    score_box = document.getElementById(
        "score"
    )

    status_box = document.getElementById(
        "status"
    )


    name = name_box.value.strip()

    score_text = score_box.value.strip()


    # ------------------------------
    # 이름 검사
    # ------------------------------

    if not name:

        status_box.innerText = (
            "이름을 입력하세요."
        )

        return


    # ------------------------------
    # 점수 검사
    # ------------------------------

    if not score_text:

        status_box.innerText = (
            "점수를 입력하세요."
        )

        return


    try:

        score = float(score_text)

    except:

        status_box.innerText = (
            "점수가 올바르지 않습니다."
        )

        return


    # ------------------------------
    # 서버로 전송
    # ------------------------------

    try:

        body = JSON.stringify({
            "name": name,
            "score": score
        })


        response = await fetch(
            "/api/submit",
            {
                "method": "POST",

                "headers": {
                    "Content-Type":
                        "application/json"
                },

                "body": body
            }
        )


        result = await response.json()


        if result["success"]:

            status_box.innerText = (
                result["message"]
            )

            name_box.value = ""

            score_box.value = ""


            # 랭킹 즉시 갱신

            await load_ranking()


        else:

            status_box.innerText = (
                result["message"]
            )


    except Exception as e:

        status_box.innerText = (
            "서버 연결에 실패했습니다."
        )


# ==========================================
# 자동 갱신
# ==========================================

async def auto_refresh():

    while True:

        await asyncio.sleep(3)

        await load_ranking()


# 최초 실행

await load_ranking()

# 자동 갱신 시작

asyncio.create_task(
    auto_refresh()
)

</script>


</body>

</html>
