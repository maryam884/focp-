#include <SFML/Graphics.hpp>
#include <SFML/Audio.hpp>
#include <iostream>
#include <vector>
#include <ctime>
#include <cmath>
#include<windows.h>
#include <chrono> 
#include <thread>
#include <atomic>
using namespace std;

//Dimensions
const int WIDTH = 15;
const int HEIGHT = 15;
const int TILE_SIZE = 40;
int score = 0;
int seconds = 40;
bool isTimeUp = false; // Flag to track if the game is paused



// Player position
float playerX = 1, playerY = 1;          // Current position
float targetX = playerX, targetY = playerY; // Target position

void Timer(int seconds) {
    seconds = 40;
    for (int i = seconds; i > 0; i--) {
        std::cout << "The time remaining is " << i << "   \r";
        std::this_thread::sleep_for(std::chrono::seconds(1));  // Wait for 1 second
    }
    cout << "Your Timer has Ended!" << endl;
    isTimeUp = true;

}


void CheckTimer(sf::RenderWindow& window) {
    // Check if the timer has run out using the global flag
    if (isTimeUp) {
        std::cout << "Time's up!" << std::endl;

        // Optionally, you can stop the game here or perform other logic
        window.close();  // Close the window after time is up
    }
}

void StartTimerInThread(int seconds) {
    // Start the timer in a separate thread
    std::thread timerThread(Timer, seconds);
    timerThread.detach();  // Detach the thread so it runs independently
}

// Level 1 layout
vector<vector<char>> maze = {
    {'#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#'},
    {'#', ' ', ' ', ' ', '#', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', '#'},
    {'#', '#', '#', ' ', '#', ' ', '#', '#', ' ', '#', ' ', '#', '#', '#', '#'},
    {'#', ' ', ' ', ' ', ' ', ' ', ' ', '#', ' ', '#', ' ', ' ', ' ', ' ', '#'},
    {'#', ' ', '#', '#', '#', '#', ' ', '#', ' ', '#', ' ', '#', '#', ' ', '#'},
    {'#', ' ', ' ', ' ', ' ', '#', ' ', '#', ' ', '#', ' ', '#', ' ', ' ', '#'},
    {'#', ' ', '#', ' ', ' ', '#', ' ', '#', ' ', '#', ' ', '#', ' ', ' ', '#'},
    {'#', ' ', '#', ' ', ' ', ' ', ' ', '#', ' ', '#', ' ', '#', '#', ' ', '#'},
    {'#', ' ', '#', ' ', ' ', '#', ' ', '#', ' ', '#', ' ', ' ', ' ', ' ', '#'},
    {'#', '#', '#', '#', '#', '#', ' ', '#', ' ', '#', ' ', '#', '#', '#', '#'},
    {'#', ' ', ' ', ' ', ' ', '#', ' ', '#', ' ', '#', ' ', ' ', ' ', ' ', '#'},
    {'#', ' ', '#', ' ', ' ', '#', ' ', '#', ' ', '#', '#', '#', '#', ' ', '#'},
    {'#', ' ', '#', ' ', ' ', ' ', ' ', '#', ' ', '#', ' ', ' ', ' ', ' ', '#'},
    {'#', ' ', '#', ' ', '#', '#', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', '#'},
    {'#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#'}
};

// Goal position
int goalX = WIDTH - 2, goalY = HEIGHT - 2;

// Sound effects
sf::SoundBuffer moveBuffer;
sf::Sound moveSound;
sf::SoundBuffer winBuffer;
sf::Sound winSound;

// Sound effects Function
void loadSounds() {
    if (!moveBuffer.loadFromFile("move.wav")) {
        cerr << "Error loading move sound!" << endl;
    }
    moveSound.setBuffer(moveBuffer);

    if (!winBuffer.loadFromFile("win.wav")) {
        cerr << "Error loading win sound!" << endl;
    }
    winSound.setBuffer(winBuffer);
}

// Draw Maze
void drawMaze(sf::RenderWindow& window) {
    window.clear(sf::Color(230, 230, 255));

    sf::RectangleShape wall(sf::Vector2f(TILE_SIZE, TILE_SIZE));
    wall.setFillColor(sf::Color(186, 85, 211));

    sf::RectangleShape goal(sf::Vector2f(TILE_SIZE, TILE_SIZE));
    goal.setFillColor(sf::Color(0, 255, 0)); // Green for goal

    sf::RectangleShape player(sf::Vector2f(TILE_SIZE, TILE_SIZE));
    player.setFillColor(sf::Color(255, 182, 193));

    // Draw maze
    for (int y = 0; y < HEIGHT; ++y) {
        for (int x = 0; x < WIDTH; ++x) {
            if (maze[y][x] == '#') {
                wall.setPosition(x * TILE_SIZE, y * TILE_SIZE);
                window.draw(wall);
            }
        }
    }

    // Draw goal
    goal.setPosition(goalX * TILE_SIZE, goalY * TILE_SIZE);
    window.draw(goal);

    // Draw player
    player.setPosition(playerX * TILE_SIZE, playerY * TILE_SIZE);
    window.draw(player);

    window.display();
}

// Move player
void movePlayer(char direction) {
    // Check if player is already moving
    if (playerX != targetX || playerY != targetY) {
        return; // Ignore input while moving
    }

    float newX = targetX;
    float newY = targetY;

    switch (direction) {
    case 'W': newY -= 1; break; // Up
    case 'S': newY += 1; break; // Down
    case 'A': newX -= 1; break; // Left
    case 'D': newX += 1; break; // Right
    }

    // Check if the move is valid
    if (newX >= 0 && newX < WIDTH && newY >= 0 && newY < HEIGHT && maze[(int)newY][(int)newX] != '#') {
        targetX = newX;
        targetY = newY;
        moveSound.play();
    }
}

// Has the player won
bool hasWon() {
    return static_cast<int>(playerX) == goalX && static_cast<int>(playerY) == goalY;


}

// Update player position towards target tile
void updatePlayerPosition(float deltaTime) {
    float speed = 5.0f; // Tiles per second
    float step = speed * deltaTime;

    if (playerX < targetX) {
        playerX += step;
        if (playerX > targetX) playerX = targetX;
    }
    else if (playerX > targetX) {
        playerX -= step;
        if (playerX < targetX) playerX = targetX;
    }

    if (playerY < targetY) {
        playerY += step;
        if (playerY > targetY) playerY = targetY;
    }
    else if (playerY > targetY) {
        playerY -= step;
        if (playerY < targetY) playerY = targetY;
    }
}
// Global variable to hold collectible positions
std::vector<std::pair<int, int>> collectiblePositions;

// Function to generate collectible positions based on level
void generateCollectibles(int level) {
    collectiblePositions.clear(); // Clear previous collectibles

    if (level == 1) {
        collectiblePositions = {
            {6,3},
            {10,8},
            {13,10},
            {11,3},
            {13,8}
        };
    }
    else if (level == 2) {
        collectiblePositions = {
            {7, 4},
            {4, 9},
            {10, 13},
            {10, 9}
        };
    }
    else if (level == 3) {
        collectiblePositions = {
            {3, 4},
            {7, 4},
            {4, 11},
            {10, 11},
            {12, 11}
        };
    }
    else if (level == 4) {
        collectiblePositions = {
            {3, 6},
            {6, 8},
            {4, 1},
            {7, 5},
            {11, 13}
        };
    }
}
void checkCollectibles(int playerX, int playerY, int& score) {
    for (auto it = collectiblePositions.begin(); it != collectiblePositions.end(); ++it) {
        if (playerX == it->first && playerY == it->second) {
            cout << "You collected a reward!" << endl;
            score += 10; // Add 10 to score
            cout << "Your Score is " << score << endl;
            collectiblePositions.erase(it); // Remove collected item
            break; // Exit the loop after collecting the reward
        }
    }
}
std::vector<std::pair<int, int>> TrapsPositions;
void generateTraps(int level) {
    TrapsPositions.clear(); // Clear previous collectibles

    if (level == 1) {
        TrapsPositions = {
            {3,1},
            {6, 13},
            {5, 7}

        };
    }
    else if (level == 2) {
        TrapsPositions = {
            {3, 10},
            {7, 5},
            {13, 4},
            {13, 8}
        };
    }
    else if (level == 3) {
        TrapsPositions = {
            {4, 6},
            {10, 8},
            {12, 3},
            {7, 12}
        };
    }
    else if (level == 4) {
        TrapsPositions = {
            {12, 10},
            {1, 7},
            {7, 9},
            {5, 8 }
        };
    }
}
void checkTraps(int playerX, int playerY, int& score) {
    for (auto it = TrapsPositions.begin(); it != TrapsPositions.end(); ++it) {
        if (playerX == it->first && playerY == it->second) {
            cout << endl << "Oops! You stepped on a Trap! " << endl;
            score -= 10; // Add 10 to score
            cout << "Your Score is " << score << endl;
            TrapsPositions.erase(it); // Remove collected item
            break; // Exit the loop after collecting the reward
        }
    }
}


// Function to generate levels
void generateNextLevel(int level) {
    if (level == 2) {
        maze = {
            {'#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#'},
            {'#', ' ', ' ', '#', ' ', ' ', '#', ' ', ' ', ' ', ' ', ' ', ' ', ' ', '#'},
            {'#', ' ', '#', '#', '#', ' ', '#', '#', ' ', '#', '#', '#', '#', ' ', '#'},
            {'#', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', '#'},
            {'#', '#', '#', ' ', '#', '#', '#', ' ', '#', '#', '#', '#', '#', ' ', '#'},
            {'#', ' ', ' ', ' ', ' ', '#', ' ', ' ', ' ', '#', ' ', ' ', ' ', ' ', '#'},
            {'#', ' ', '#', '#', ' ', '#', ' ', '#', ' ', '#', '#', '#', '#', ' ', '#'},
            {'#', ' ', '#', ' ', ' ', '#', ' ', '#', ' ', ' ', ' ', ' ', ' ', ' ', '#'},
            {'#', ' ', '#', ' ', ' ', '#', '#', '#', '#', '#', ' ', '#', '#', ' ', '#'},
            {'#', ' ', '#', '#', ' ', ' ', ' ', '#', ' ', '#', ' ', '#', ' ', ' ', '#'},
            {'#', ' ', ' ', ' ', ' ', '#', ' ', '#', ' ', ' ', ' ', '#', ' ', ' ', '#'},
            {'#', ' ', '#', '#', ' ', '#', '#', '#', '#', '#', ' ', '#', ' ', '#', '#'},
            {'#', ' ', '#', ' ', ' ', ' ', ' ', ' ', ' ', '#', ' ', ' ', ' ', ' ', '#'},
            {'#', ' ', '#', ' ', '#', '#', ' ', ' ', ' ', ' ', ' ', '#', ' ', ' ', '#'},
            {'#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#'}
        };

    }
    else if (level == 3) {
        maze = {
            {'#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#'},
            {'#', ' ', ' ', ' ', ' ', ' ', '#', ' ', ' ', ' ', ' ', ' ', ' ', ' ', '#'},
            {'#', '#', '#', '#', '#', ' ', '#', '#', ' ', '#', '#', '#', '#', ' ', '#'},
            {'#', ' ', '#', ' ', ' ', ' ', '#', ' ', ' ', ' ', ' ', '#', ' ', ' ', '#'},
            {'#', ' ', '#', ' ', '#', '#', '#', ' ', '#', '#', '#', '#', '#', ' ', '#'},
            {'#', ' ', ' ', ' ', '#', ' ', ' ', ' ', ' ', ' ', ' ', ' ', '#', '#', '#'},
            {'#', ' ', '#', '#', '#', ' ', '#', '#', '#', '#', '#', ' ', '#', ' ', '#'},
            {'#', ' ', '#', ' ', ' ', ' ', '#', ' ', ' ', '#', ' ', ' ', ' ', ' ', '#'},
            {'#', ' ', ' ', ' ', '#', '#', '#', '#', '#', '#', '#', '#', '#', ' ', '#'},
            {'#', '#', '#', ' ', ' ', ' ', ' ', '#', ' ', '#', ' ', '#', '#', ' ', '#'},
            {'#', ' ', ' ', ' ', '#', ' ', ' ', '#', ' ', '#', ' ', ' ', ' ', ' ', '#'},
            {'#', ' ', '#', '#', '#', '#', ' ', '#', ' ', '#', '#', '#', ' ', ' ', '#'},
            {'#', ' ', '#', ' ', ' ', '#', ' ', ' ', '#', ' ', ' ', '#', ' ', '#', '#'},
            {'#', ' ', '#', ' ', '#', '#', '#', ' ', '#', '#', ' ', ' ', ' ', ' ', '#'},
            {'#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#'}
        };

    }
    else if (level == 4) {
        maze = {
            {'#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#'},
            {'#', ' ', ' ', '#', ' ', ' ', ' ', ' ', ' ', '#', '#', '#', ' ', ' ', '#'},
            {'#', '#', ' ', '#', ' ', '#', '#', '#', ' ', ' ', ' ', '#', '#', ' ', '#'},
            {'#', ' ', ' ', ' ', ' ', ' ', ' ', '#', ' ', '#', ' ', ' ', ' ', ' ', '#'},
            {'#', ' ', '#', '#', '#', '#', ' ', '#', ' ', '#', '#', '#', '#', '#', '#'},
            {'#', ' ', '#', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', '#'},
            {'#', ' ', '#', ' ', '#', '#', '#', ' ', '#', ' ', '#', '#', '#', ' ', '#'},
            {'#', ' ', ' ', ' ', ' ', ' ', '#', ' ', '#', ' ', ' ', ' ', '#', ' ', '#'},
            {'#', '#', '#', '#', '#', ' ', ' ', ' ', '#', '#', '#', ' ', '#', '#', '#'},
            {'#', ' ', ' ', ' ', '#', ' ', '#', ' ', ' ', ' ', '#', ' ', ' ', '#', '#'},
            {'#', ' ', '#', '#', '#', ' ', '#', '#', '#', ' ', '#', '#', '#', ' ', '#'},
            {'#', ' ', '#', ' ', ' ', ' ', ' ', ' ', '#', ' ', '#', ' ', ' ', ' ', '#'},
            {'#', ' ', '#', ' ', '#', '#', '#', '#', '#', '#', '#', '#', ' ', ' ', '#'},
            {'#', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', '#'},
            {'#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#'}
        };
    }
    generateCollectibles(level);
    generateTraps(level);
 


}


// Main game function
int main() {
    srand(static_cast<unsigned>(time(0)));

    sf::RenderWindow window(sf::VideoMode(WIDTH * TILE_SIZE, HEIGHT * TILE_SIZE), "Maze Game");
    loadSounds();

    int level = 1;
    StartTimerInThread(seconds);
    generateNextLevel(level);


    sf::Clock clock;
    while (window.isOpen()) {
        sf::Event event;
        while (window.pollEvent(event)) {
            if (event.type == sf::Event::Closed) {
                window.close();
            }
        }

        // Handle movement input
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::W)) {
            movePlayer('W');
        }
        else if (sf::Keyboard::isKeyPressed(sf::Keyboard::S)) {
            movePlayer('S');
        }
        else if (sf::Keyboard::isKeyPressed(sf::Keyboard::A)) {
            movePlayer('A');
        }
        else if (sf::Keyboard::isKeyPressed(sf::Keyboard::D)) {
            movePlayer('D');
        }

        // Update player position
        float deltaTime = clock.restart().asSeconds();
        updatePlayerPosition(deltaTime);
        static int score = 0;
        checkCollectibles(static_cast<int>(playerX), static_cast<int>(playerY), score);
        checkTraps(static_cast<int>(playerX), static_cast<int>(playerY), score);
        CheckTimer(window);

        // Check for win condition
        if (hasWon()) {
            winSound.play();
            score = 0;


            cout << "You won Level " << level << "!" << endl;
            cout << "You scored " << score << " in level " << level << endl<<endl;
            level++;
            cout << "You are at Level  " << level << "!" << endl;
            cout << "Your current Score is 0." << endl;
            if (level > 4) {
                cout << "Congratulations! You've completed all levels!" << endl;
                break;
            }
            generateNextLevel(level);
            playerX = 1;
            playerY = 1;
            targetX = 1;
            targetY = 1;
        }

        // Render the maze
        drawMaze(window);
    }

    return 0;
}



[README-2.md](https://github.com/user-attachments/files/18245733/README-2.md)
