# RedLightGreenLight
==================================
bangtal library를 사용한 '무궁화 꽃이 피었습니다' 게임 프로젝트입니다.

인형이 뒤를 돌아본 순간에 플레이버튼을 클릭하여 결승선을 통과하면 됩니다.

![readme_image](https://user-images.githubusercontent.com/90536655/137574938-2b4ba25c-84f9-49e6-9773-38a266d425a9.png)

#include <bangtal>
using namespace bangtal;

int main() {
	setGameOption(GameOption::GAME_OPTION_INVENTORY_BUTTON, false);
	setGameOption(GameOption::GAME_OPTION_MESSAGE_BOX_BUTTON, false);

	auto scene1 = Scene::create("무궁화 꽃이 피었습니다", "Images/배경1.png");
	auto scene2 = Scene::create("무궁화 꽃이 피었습니다", "Images/배경2.png");
	showMessage("<<<<무궁화 꽃이 피었습니다 룰 설명>>>> \n화살표 버튼을 눌러 이동할 수 있다.\n인형이 돌아본 순간 움직이면 실패!\n인형에게 걸리지 않고 결승선을 통과하면 성공!");

	auto start = Object::create("Images/start.png", scene1, 590, 360);
	auto end = Object::create("Images/end.png", scene1, 590, 300, false);
	auto doll = Object::create("Images/doll2.png", scene2, 1080, 150);
	auto play = Object::create("Images/play.png", scene2, 590, 80);
	auto playerX = 10, playerY = 150;
	auto player = Object::create("Images/player2.png", scene2, playerX, playerY);
	doll->setScale(0.5f);
	player->setScale(0.2f);

	bool player_check = false;                   // 플레이어의 움직임에 따라 이미지를 변경할 상태
	bool state = false;                          // 인형이 뒤를 돌아본 순간을 나타낼 상태

	auto timer1 = Timer::create(3.f);            // 인형이 뒤를 볼 시간
	auto timer2 = Timer::create(2.f);            // 인형이 앞을 볼 시간
	timer1->setOnTimerCallback([&](TimerPtr)->bool {
		state = true;
		doll->setImage("Images/doll1.png");
		timer2->set(2.f);
		timer2->start();

		return true;
		});

	timer2->setOnTimerCallback([&](TimerPtr)->bool {
		state = false;
		doll->setImage("Images/doll2.png");
		timer1->set(3.f);
		timer1->start();

		return true;
		});

	start->setOnMouseCallback([&](ObjectPtr object, int x, int y, MouseAction action)->bool {
		scene2->enter();
		play->show();
		timer1->set(3.f);
		timer1->start();

		playerX = 10;
		player->locate(scene2, playerX, playerY);

		showMessage("무궁화 꽃이 피었습니다~");

		return true;
		});

	play->setOnMouseCallback([&](ObjectPtr object, int x, int y, MouseAction action)->bool {
		playerX = playerX + 15;
		player->locate(scene2, playerX, playerY);

		if (state == false) {                                     // 인형이 뒤를 본 순간에 플레이어의 움직임에 따라 이미지 변경
			if (player_check == false) {
				player->setImage("Images/player1.png");
				playerX = playerX + 20;
				player->locate(scene2, playerX, playerY);
				player_check = true;
			}
			else {
				player->setImage("Images/player2.png");
				playerX = playerX + 20;
				player->locate(scene2, playerX, playerY);
				player_check = false;
			}
		}

		else {                                                   // 인형이 앞을 본 순간에 플레이어가 움직이면 게임 종료
			timer1->stop();
			timer2->stop();
			showMessage("미션 실패");

			scene1->enter();
			play->hide();
			start->show();
			end->show();
		}


		if (playerX > 1050) {                                   // 결승선을 통과하면 게임 성공
			timer1->stop();
			timer2->stop();

			showMessage("미션 성공!");

			scene1->enter();
			play->hide();
			start->show();
			end->show();
		}


		return true;
		});

	end->setOnMouseCallback([&](ObjectPtr object, int x, int y, MouseAction action)->bool {
		endGame();
		return true;
		});

	startGame(scene1);
	return 0;
}
