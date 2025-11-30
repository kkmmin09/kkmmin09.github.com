숫자 맞추기 게임

난이도
1. 쉬움(1~10)
2. 보통(1~50)
3. 어려움(1~100)
정답을 맞출 때까지 계속 입려받도록 코드를 만들었습니다.

import random # 랜덤한 숫자를 만들어낸다.

print("난이도 선택")
print("1. 1~10")
print("2. 1~50")
print("3. 1~100")

d = int(input("난이도: "))

if d == 1:
    end = 10
elif d == 2:
    end = 50
else:
    end = 100
    
# 선택한 숫자에 따라 숫자의 범위를 결정합니다.

answer = random.randint(1, end)
# 1부터 end까지의 숫자중 하나를 골라 정답으로 설정합니다.

print("1부터 " + str(end) + " 사이 숫자 맞춰보세요")
# 범위를 알려주고 게임을 시작합니다.

cnt = 0
# 시도한 횟수를 세기 위한 변수입니다.

while True:
    try:
        x = int(input("숫자 입력: "))
        cnt = cnt + 1

        if x < answer:
            print("작음")
        elif x > answer:
            print("큼")
        else:
            print("정답 " + str(cnt) + "번 만에 맞춤")
            break
# 숫자를 맞출때까지 무한반복으로 숫자를 입력 받습니다. 만약 틀린다면 cnt 변수의 숫자가 증가합니다.
    except:
        print("숫자를 쓰세요")
      # 만약 숫자가 아닌 것을 입력했을때 출력됩니다.


