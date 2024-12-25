GitHub Repository: [https://github.com/maryam884/focp-.git]

README.md Overview:
1. Project Title: Maze Game Project
2. Group Members:
   - Armish Saleem
   - Maryam Asif
   - Munha Shahzad
   - Fatima Ahmad
3. Short Project Description:
   A maze game built using SFML where players navigate through challenging mazes, collect items, avoid traps, and beat timers to progress through levels.  The main goal is to excape from the maze.
4. Features:
   - Randomly generated mazes.
   - Timer and traps for added difficulty.
   - Increasing complexity with progression.
   - Collectibles to boost scores.
5. Technical Architecture:
   - Custom Modules: Input Handling, Game Logic, Maze Generation, Trap Logic, etc.
   - Third-Party Libraries: SFML for graphics, input, and audio.
6. How to Run and Deploy:
   - Clone the repository:
     `git clone [https://github.com/maryam884/focp-.git]`
   - Install SFML: Follow the instructions on [SFML's website](https://www.sfml-dev.org).
   - Compile the project using a C++ compiler with SFML configured.
   - Run the executable file to start the game.
   - DETAILED EXPLANATION
Clone the Repository:
Open a terminal or command prompt.
Use the following command to clone your repository to your local machine:
git clone [https://github.com/maryam884/focp-.git]

Install SFML Library:
SFML is required for this project to handle graphics, input, and sound. Follow these steps:
Go to the SFML official website and download the appropriate version for your operating system.
Install the library by following the platform-specific instructions provided on the SFML website:
Windows: Use precompiled binaries or set up SFML manually with Visual Studio.
Mac: Install SFML using Homebrew:
brew install sfml
Linux: Install SFML using your package manager:
sudo apt-get install libsfml-dev
Open the Project in Your IDE:
Open your favorite C++ IDE (e.g., Visual Studio, Code::Blocks, or CLion).
Configure the project to include the SFML library. Ensure that:
Include directories point to the SFML include folder.
Library directories point to the SFML lib folder.
Link the required SFML libraries (e.g., sfml-graphics, sfml-audio, sfml-system, sfml-window).
Compile the Project:
Build the project in your IDE or use a compiler directly from the command line. For example:
g++ -o MazeGame main.cpp -lsfml-graphics -lsfml-window -lsfml-system -lsfml-audio
Make sure all the source files are included in the compilation.
Run the Game:
Once compiled, execute the generated binary or .exe file to start the game. Example:
./MazeGame
On Windows, double-click the .exe file.
Adjust Game Settings (if needed):
Modify any configuration files (if available) for game settings like difficulty, number of lives, or maze size.
Deploy the Game (Optional):
Package the game files, including:
The executable or binary file.
Required SFML libraries (e.g., .dll files on Windows).
Assets like sound, images, or configuration files.
Share the packaged game with others or upload it to a platform like itch.io for distribution.
By following these steps, you'll be able to clone your repository, install dependencies, compile, and run the game.
