# SFML 활용 컴퓨터 그래픽스 개별 과제
## SFML Tennis 코드를 개선하기

 SFML 활용 컴퓨터 그래픽스 개별 과제 안내서

<details>
 
 <summary>개별 과제 안내서 상세 내용</summary>
SFML Tennis 코드를 개선

2가지 이상 아이디어로 게임 프로그램을 개선 하시오.

[제출]
코드는 Github에 제출하고,

캡쳐한 화면도 제출해 주세요.(첨부파일 사절 -1)
개선한 내용을 글로 적어 제출하면 됩니다.

채점
1. 제출시 3점
2. 아이디어의 창의성을 평가합니다. 2점

</details>

###  코드

```
////////////////////////////////////////////////////////////
// Headers
////////////////////////////////////////////////////////////
#include <SFML/Graphics.hpp>
#include <SFML/Audio.hpp>
#include <cmath>
#include <ctime>
#include <cstdlib>
#include <vector>

#ifdef SFML_SYSTEM_IOS
#include <SFML/Main.hpp>
#endif

std::string resourcesDir()
{
#ifdef SFML_SYSTEM_IOS
    return "";
#else
    return "resources/";
#endif
}

struct Ball {
    sf::CircleShape shape;
    float angle;
    float speed;

    Ball(float radius, float initSpeed, float initAngle)
        : angle(initAngle), speed(initSpeed) {
        shape.setRadius(radius - 3);
        shape.setOutlineThickness(2);
        shape.setOutlineColor(sf::Color::Black);
        shape.setFillColor(sf::Color::White);
        shape.setOrigin(radius / 2, radius / 2);
    }
};

////////////////////////////////////////////////////////////
/// Entry point of application
///
/// \return Application exit code
///
////////////////////////////////////////////////////////////
int main()
{
    std::srand(static_cast<unsigned int>(std::time(NULL)));

    // Define some constants
    const float pi = 3.14159f;
    const float gameWidth = 800;
    const float gameHeight = 600;
    sf::Vector2f paddleSize(25, 100);
    float ballRadius = 10.f;

    // Create the window of the application
    sf::RenderWindow window(sf::VideoMode(static_cast<unsigned int>(gameWidth), static_cast<unsigned int>(gameHeight), 32), "SFML Tennis",
                            sf::Style::Titlebar | sf::Style::Close);
    window.setVerticalSyncEnabled(true);

    // Load the sounds used in the game
    sf::SoundBuffer ballSoundBuffer;
    if (!ballSoundBuffer.loadFromFile(resourcesDir() + "ball.wav"))
        return EXIT_FAILURE;
    sf::Sound ballSound(ballSoundBuffer);

    // Create the SFML logo texture:
    sf::Texture sfmlLogoTexture;
    if(!sfmlLogoTexture.loadFromFile(resourcesDir() + "sfml_logo.png"))
        return EXIT_FAILURE;
    sf::Sprite sfmlLogo;
    sfmlLogo.setTexture(sfmlLogoTexture);
    sfmlLogo.setPosition(170, 50);

    // Create the left paddle
    sf::RectangleShape leftPaddle;
    leftPaddle.setSize(paddleSize - sf::Vector2f(3, 3));
    leftPaddle.setOutlineThickness(3);
    leftPaddle.setOutlineColor(sf::Color::Black);
    leftPaddle.setFillColor(sf::Color(100, 100, 200));
    leftPaddle.setOrigin(paddleSize / 2.f);

    // Create the right paddle
    sf::RectangleShape rightPaddle;
    rightPaddle.setSize(paddleSize - sf::Vector2f(3, 3));
    rightPaddle.setOutlineThickness(3);
    rightPaddle.setOutlineColor(sf::Color::Black);
    rightPaddle.setFillColor(sf::Color(200, 100, 100));
    rightPaddle.setOrigin(paddleSize / 2.f);

    // Load the text font
    sf::Font font;
    if (!font.loadFromFile(resourcesDir() + "tuffy.ttf"))
        return EXIT_FAILURE;

    // Initialize the pause message
    sf::Text pauseMessage;
    pauseMessage.setFont(font);
    pauseMessage.setCharacterSize(40);
    pauseMessage.setPosition(170.f, 200.f);
    pauseMessage.setFillColor(sf::Color::White);

    #ifdef SFML_SYSTEM_IOS
    pauseMessage.setString("Welcome to SFML Tennis!\nTouch the screen to start the game.");
    #else
    pauseMessage.setString("Welcome to SFML Tennis!\n\nPress space to start the game.");
    #endif

    // Initialize the score
    int leftScore = 0;
    int rightScore = 0;
    sf::Text leftScoreText;
    leftScoreText.setFont(font);
    leftScoreText.setCharacterSize(30);
    leftScoreText.setPosition(200.f, 20.f);
    leftScoreText.setFillColor(sf::Color::White);

    sf::Text rightScoreText;
    rightScoreText.setFont(font);
    rightScoreText.setCharacterSize(30);
    rightScoreText.setPosition(600.f, 20.f);
    rightScoreText.setFillColor(sf::Color::White);

    // Define the paddles properties
    sf::Clock AITimer;
    const sf::Time AITime   = sf::seconds(0.05f); // Decrease AI reaction time
    const float paddleSpeed = 400.f;
    float rightPaddleSpeed  = 0.f;

    std::vector<Ball> balls;
    float initialBallSpeed = 400.f;

    sf::Clock clock;
    bool isPlaying = false;
    while (window.isOpen())
    {
        // Handle events
        sf::Event event;
        while (window.pollEvent(event))
        {
            // Window closed or escape key pressed: exit
            if ((event.type == sf::Event::Closed) ||
               ((event.type == sf::Event::KeyPressed) && (event.key.code == sf::Keyboard::Escape)))
            {
                window.close();
                break;
            }

            // Space key pressed: play
            if (((event.type == sf::Event::KeyPressed) && (event.key.code == sf::Keyboard::Space)) ||
                (event.type == sf::Event::TouchBegan))
            {
                if (!isPlaying)
                {
                    // (re)start the game
                    isPlaying = true;
                    clock.restart();

                    // Reset the position of the paddles and ball
                    leftPaddle.setPosition(10.f + paddleSize.x / 2.f, gameHeight / 2.f);
                    rightPaddle.setPosition(gameWidth - 10.f - paddleSize.x / 2.f, gameHeight / 2.f);

                    // Reset balls
                    balls.clear();
                    balls.emplace_back(ballRadius, initialBallSpeed, static_cast<float>(std::rand() % 360) * 2.f * pi / 360.f);
                    balls[0].shape.setPosition(gameWidth / 2.f, gameHeight / 2.f);

                    // Make sure the initial ball angle is not too much vertical
                    while (std::abs(std::cos(balls[0].angle)) < 0.7f) {
                        balls[0].angle = static_cast<float>(std::rand() % 360) * 2.f * pi / 360.f;
                    }
                }
            }

            // Window size changed, adjust view appropriately
            if (event.type == sf::Event::Resized)
            {
                sf::View view;
                view.setSize(gameWidth, gameHeight);
                view.setCenter(gameWidth / 2.f, gameHeight / 2.f);
                window.setView(view);
            }
        }

        if (isPlaying)
        {
            float deltaTime = clock.restart().asSeconds();

            // Move the player's paddle
            if (sf::Keyboard::isKeyPressed(sf::Keyboard::Up) &&
               (leftPaddle.getPosition().y - paddleSize.y / 2 > 5.f))
            {
                leftPaddle.move(0.f, -paddleSpeed * deltaTime);
            }
            if (sf::Keyboard::isKeyPressed(sf::Keyboard::Down) &&
               (leftPaddle.getPosition().y + paddleSize.y / 2 < gameHeight - 5.f))
            {
                leftPaddle.move(0.f, paddleSpeed * deltaTime);
            }

            if (sf::Touch::isDown(0))
            {
                sf::Vector2i pos = sf::Touch::getPosition(0);
                sf::Vector2f mappedPos = window.mapPixelToCoords(pos);
                leftPaddle.setPosition(leftPaddle.getPosition().x, mappedPos.y);
            }

            // Move the computer's paddle for each ball
            if (AITimer.getElapsedTime() > AITime)
            {
                AITimer.restart();

                float closestBallY = balls[0].shape.getPosition().y;
                for (const auto& ball : balls) {
                    if (std::abs(ball.shape.getPosition().x - rightPaddle.getPosition().x) < std::abs(closestBallY - rightPaddle.getPosition().y)) {
                        closestBallY = ball.shape.getPosition().y;
                    }
                }
                rightPaddleSpeed = (closestBallY - rightPaddle.getPosition().y) * 10.f;

                for (const auto& ball : balls) {
                    if (((rightPaddleSpeed < 0.f) && (rightPaddle.getPosition().y - paddleSize.y / 2 > 5.f)) ||
                        ((rightPaddleSpeed > 0.f) && (rightPaddle.getPosition().y + paddleSize.y / 2 < gameHeight - 5.f))) {
                        rightPaddle.move(0.f, rightPaddleSpeed * deltaTime);
                    }
                }
            }

            for (auto& ball : balls) {
                // Move the ball
                float factor = ball.speed * deltaTime;
                ball.shape.move(std::cos(ball.angle) * factor, std::sin(ball.angle) * factor);

                #ifdef SFML_SYSTEM_IOS
                const std::string inputString = "Touch the screen to restart.";
                #else
                const std::string inputString = "Press space to restart or\nescape to exit.";
                #endif

                // Check collisions between the ball and the screen
                if (ball.shape.getPosition().x - ballRadius < 0.f)
                {
                    isPlaying = false;
                    pauseMessage.setString("You Lost!\n\n" + inputString);
                }
                if (ball.shape.getPosition().x + ballRadius > gameWidth)
                {
                    isPlaying = false;
                    pauseMessage.setString("You Won!\n\n" + inputString);
                }
                if (ball.shape.getPosition().y - ballRadius < 0.f)
                {
                    ballSound.play();
                    ball.angle = -ball.angle;
                    ball.shape.setPosition(ball.shape.getPosition().x, ballRadius + 0.1f);
                }
                if (ball.shape.getPosition().y + ballRadius > gameHeight)
                {
                    ballSound.play();
                    ball.angle = -ball.angle;
                    ball.shape.setPosition(ball.shape.getPosition().x, gameHeight - ballRadius - 0.1f);
                }

                // Check the collisions between the ball and the paddles
                // Left Paddle
                if (ball.shape.getPosition().x - ballRadius < leftPaddle.getPosition().x + paddleSize.x / 2 &&
                    ball.shape.getPosition().x - ballRadius > leftPaddle.getPosition().x &&
                    ball.shape.getPosition().y + ballRadius >= leftPaddle.getPosition().y - paddleSize.y / 2 &&
                    ball.shape.getPosition().y - ballRadius <= leftPaddle.getPosition().y + paddleSize.y / 2)
                {
                    if (ball.shape.getPosition().y > leftPaddle.getPosition().y)
                        ball.angle = pi - ball.angle + static_cast<float>(std::rand() % 20) * pi / 180;
                    else
                        ball.angle = pi - ball.angle - static_cast<float>(std::rand() % 20) * pi / 180;

                    ballSound.play();
                    ball.shape.setPosition(leftPaddle.getPosition().x + ballRadius + paddleSize.x / 2 + 0.1f, ball.shape.getPosition().y);

                    // Increase the ball speed each time it hits the paddle
                    ball.speed += 10.f;

                    // Add a new ball
                    float newAngle;
                    do
                    {
                        newAngle = static_cast<float>(std::rand() % 360) * 2.f * pi / 360.f;
                    }
                    while (std::abs(std::cos(newAngle)) < 0.7f);
                    balls.emplace_back(ballRadius, initialBallSpeed, newAngle);
                    balls.back().shape.setPosition(gameWidth / 2.f, gameHeight / 2.f);
                }

                // Right Paddle
                if (ball.shape.getPosition().x + ballRadius > rightPaddle.getPosition().x - paddleSize.x / 2 &&
                    ball.shape.getPosition().x + ballRadius < rightPaddle.getPosition().x &&
                    ball.shape.getPosition().y + ballRadius >= rightPaddle.getPosition().y - paddleSize.y / 2 &&
                    ball.shape.getPosition().y - ballRadius <= rightPaddle.getPosition().y + paddleSize.y / 2)
                {
                    if (ball.shape.getPosition().y > rightPaddle.getPosition().y)
                        ball.angle = pi - ball.angle + static_cast<float>(std::rand() % 20) * pi / 180;
                    else
                        ball.angle = pi - ball.angle - static_cast<float>(std::rand() % 20) * pi / 180;

                    ballSound.play();
                    ball.shape.setPosition(rightPaddle.getPosition().x - ballRadius - paddleSize.x / 2 - 0.1f, ball.shape.getPosition().y);

                    // Increase the ball speed each time it hits the paddle
                    ball.speed += 10.f;

                    // Add a new ball
                    float newAngle;
                    do
                    {
                        newAngle = static_cast<float>(std::rand() % 360) * 2.f * pi / 360.f;
                    }
                    while (std::abs(std::cos(newAngle)) < 0.7f);
                    balls.emplace_back(ballRadius, initialBallSpeed, newAngle);
                    balls.back().shape.setPosition(gameWidth / 2.f, gameHeight / 2.f);
                }
            }

            // Update the score text
            leftScoreText.setString(std::to_string(leftScore));
            rightScoreText.setString(std::to_string(rightScore));
        }

        // Clear the window
        window.clear(sf::Color(50, 50, 50));

        if (isPlaying)
        {
            // Draw the paddles, the ball, and the score
            window.draw(leftPaddle);
            window.draw(rightPaddle);
            for (const auto& ball : balls)
                window.draw(ball.shape);
            window.draw(leftScoreText);
            window.draw(rightScoreText);
        }
        else
        {
            // Draw the pause message
            window.draw(pauseMessage);
            window.draw(sfmlLogo);
        }

        // Display things on screen
        window.display();
    }

    return EXIT_SUCCESS;
}

```
### p5.js에서의 코드 실행 결과
---
![캡처2](https://github.com/rex6928/Computer-Graphic3/assets/162276764/c18caa42-1051-4b6a-82c6-98ba287c8af3)
---
![캡처1](https://github.com/rex6928/Computer-Graphic3/assets/162276764/4d32ebcf-ec20-42d7-b847-235926182aba)
---

### 소감

이번 과제를 통해 SFML을 사용한 2D 게임 개발의 기초를 익힐 수 있었습니다. 공이 패들에 맞을 때마다 공의 개수가 증가하는 기능을 구현하면서, 충돌 감지와 물리 계산의 중요성을 깨달았습니다. 또한, AI의 움직임을 개선하여 모든 공에 반응하도록 하면서 게임 로직의 복잡성을 다루는 방법을 배웠습니다. 코드를 반복적으로 수정하고 테스트하는 과정에서 디버깅 능력도 향상되었습니다. 이 과제를 통해 얻은 경험이 앞으로의 게임 개발에 큰 도움이 될 것이라 확신합니다.















