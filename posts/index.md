---
layout: post
title: "PyQt5 + MySQL 뉴스 조회 프로그램 코드 분석"
excerpt: "감성 분석 기능이 포함된 뉴스 조회 GUI 프로그램 코드를 한 줄씩 뜯어봤습니다."
comments: true
---

# PyQt5 + MySQL 뉴스 조회 프로그램 코드 분석

날짜와 카테고리를 선택하면 MySQL DB에서 뉴스 기사를 조회하고, 감성 사전을 이용해 각 기사의 감성 점수를 계산해 테이블에 보여주는 PyQt5 프로그램입니다. 제목을 더블클릭하면 원본 기사 URL이 브라우저로 열립니다.

아래에서 코드를 부분별로 나눠 설명합니다.

---

```python
import sys
import json
import pymysql
import webbrowser

from PyQt5 import uic
```
프로그램 실행, JSON 처리, MySQL 연결, 웹브라우저 열기, PyQt5 UI 로딩에 필요한 모듈들을 불러옵니다.

```python
from PyQt5.QtCore import Qt
from PyQt5.QtWidgets import (
    QApplication,
    QWidget,
    QTableWidgetItem,
    QMessageBox
)
```
PyQt5에서 사용할 UI 관련 클래스들(창, 테이블 아이템, 메시지박스 등)을 가져옵니다.

```python
UI_FILE = r"C:\Users\user\Desktop\ㄱㄱㄱㅁui.ui"
SENTIMENT_FILE = r"C:\Users\user\Desktop\sentiment.json"
```
QtDesigner로 만든 `.ui` 파일 경로와, 감성 분석용 단어 사전 파일 경로를 지정합니다.

```python
with open(
    SENTIMENT_FILE,
    "r",
    encoding="utf-8"
) as f:
    sentiment_dict = json.load(f)
```
감성 사전 JSON 파일을 읽어 `sentiment_dict`에 저장합니다. (단어와 극성 점수 목록)

```python
conn = pymysql.connect(
    host="183.100.182.169",
    user="root",
    password="swhacademy!",
    db="KimKyoungMin",
    port=3306,
    charset="utf8"
)
```
MySQL 서버에 접속합니다. (호스트, 계정, 비밀번호, DB명 지정)

```python
cursor = conn.cursor()
```
SQL 쿼리를 실행하기 위한 커서 객체를 생성합니다.

```python
category_map = {
    "정치": 100,
    "경제": 101,
    "사회": 102,
    "생활/문화": 103,
    "세계": 104,
    "IT/과학": 105,
    "연예": 106,
    "속보": 107,
    "오피니언": 110
}
```
화면에 표시되는 한글 카테고리명을 DB에 저장된 카테고리 코드(숫자)로 변환하기 위한 매핑 테이블입니다.

```python
def analyze_sentiment(content):
    if not content:
        return 0
    score = 0
```
감성 분석 함수 시작. 내용이 없으면 0점을 반환하고, 점수를 담을 변수를 초기화합니다.

```python
    for item in sentiment_dict:
        word_root = item["word_root"]
        polarity = int(item["polarity"])
        if word_root in content:
            score += polarity
    return score
```
사전의 모든 단어를 순회하며, 기사 본문에 해당 단어가 포함되어 있으면 극성 점수를 누적합니다. 최종 점수를 반환합니다.

```python
def search_news(date, category_name):
    if category_name not in category_map:
        return []
    category_code = category_map[category_name]
```
뉴스 검색 함수 시작. 카테고리명이 유효하지 않으면 빈 리스트 반환, 유효하면 코드로 변환합니다.

```python
    sql = """
    SELECT
        title,
        content,
        url,
        page,
        created_at
    FROM articles
    WHERE DATE(created_at) = %s
      AND category = %s
    ORDER BY page ASC, id DESC
    """
```
지정한 날짜와 카테고리에 맞는 기사를 조회하는 SQL 쿼리문입니다.

```python
    cursor.execute(
        sql,
        (
            date,
            category_code
        )
    )
    return cursor.fetchall()
```
쿼리를 실행하고 결과 전체를 가져와 반환합니다.

```python
class MyWindow(QWidget):
    def __init__(self):
        super().__init__()
        uic.loadUi(
            UI_FILE,
            self
        )
```
메인 윈도우 클래스 정의. `.ui` 파일을 불러와 화면을 구성합니다.

```python
        self.pushButton.clicked.connect(
            self.search_button_clicked
        )
        self.tableWidget.cellDoubleClicked.connect(
            self.title_double_clicked
        )
```
검색 버튼 클릭 이벤트와, 테이블 셀 더블클릭 이벤트에 각각 함수를 연결(시그널-슬롯)합니다.

```python
    def search_button_clicked(self):
        try:
            date = self.dateEdit.date().toString(
                "yyyy-MM-dd"
            )
            category = self.comboBox.currentText()
```
검색 버튼 클릭 시 실행되는 함수 시작. 날짜 선택 위젯과 콤보박스에서 값을 읽어옵니다.

```python
            results = search_news(
                date,
                category
            )
            self.tableWidget.setRowCount(0)
```
입력받은 날짜/카테고리로 뉴스를 검색하고, 기존 테이블 내용을 초기화합니다.

```python
            if not results:
                QMessageBox.information(
                    self,
                    "조회 결과",
                    "해당 날짜와 카테고리의 뉴스가 없습니다."
                )
                return
```
검색 결과가 없으면 안내 메시지 창을 띄우고 함수를 종료합니다.

```python
            for row_data in results:
                title = row_data[0]
                content = row_data[1]
                url = row_data[2]
                page = row_data[3]
                created_at = row_data[4]
```
검색된 기사들을 하나씩 순회하며 제목, 본문, URL, 페이지, 작성일을 꺼냅니다.

```python
                sentiment_score = analyze_sentiment(
                    content
                )
                row = self.tableWidget.rowCount()
                self.tableWidget.insertRow(
                    row
                ) 
```
기사 본문의 감성 점수를 계산하고, 테이블에 새로운 행을 추가합니다.

```python
                title_item = QTableWidgetItem(
                    str(title)
                )
                title_item.setData(
                    Qt.UserRole, 
                    url
                )
```
제목을 테이블 아이템으로 만들고, 화면에는 안 보이지만 나중에 꺼낼 수 있도록 URL을 `UserRole`에 숨겨서 저장합니다.

```python
                self.tableWidget.setItem(
                    row,
                    0,
                    title_item
                )
                self.tableWidget.setItem(
                    row,
                    1,
                    QTableWidgetItem(str(category))
                )
```
0번 열에 제목, 1번 열에 카테고리를 표시합니다.

```python
                self.tableWidget.setItem(
                    row,
                    2,
                    QTableWidgetItem(str(created_at))
                )
                self.tableWidget.setItem(
                    row,
                    3,
                    QTableWidgetItem(str(sentiment_score))
                )
```
2번 열에 작성일, 3번 열에 감성 점수를 표시합니다.

```python
            self.tableWidget.resizeColumnsToContents()
        except Exception as e:
            print("조회 오류:", e)
            QMessageBox.critical(self, "오류", str(e))
```
모든 행 추가가 끝나면 열 너비를 내용에 맞게 조정합니다. 오류 발생 시 콘솔에 출력하고 오류 메시지 창을 띄웁니다.

```python
    def title_double_clicked(self, row, column):
        if column != 0: 
            return
        item = self.tableWidget.item(row, column)
        if item is None: 
            return
```
테이블 셀 더블클릭 시 실행되는 함수. 제목 열(0번)이 아니거나 아이템이 없으면 그냥 종료합니다.

```python
        url = item.data(Qt.UserRole)
        if url: 
            webbrowser.open(str(url))
        else:
            QMessageBox.warning(
                self,
                "URL 오류",
                "해당 기사에 저장된 URL이 없습니다."
            )
```
숨겨둔 URL을 꺼내서 있으면 브라우저로 열고, 없으면 경고창을 띄웁니다.

```python
if __name__ == "__main__":
    try:
        app = QApplication(sys.argv)
        window = MyWindow()
        window.show()
        sys.exit(app.exec_())
```
프로그램의 진입점. Qt 애플리케이션을 만들고, 창을 띄운 뒤 이벤트 루프를 실행합니다.

```python
    finally:
        cursor.close()
        conn.close()
```
프로그램이 종료될 때 (정상 종료든 오류든) DB 커서와 연결을 안전하게 닫습니다.

---
