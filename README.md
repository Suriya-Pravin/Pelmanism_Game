# Pelmanism_Game
## Aim:
This code implements a memory-matching game, also known as "Pelmanism" or simply a memory game. In this version, the game allows a player to compete against an AI to find all matching pairs of animal images on a 4x4 grid. Here's a breakdown of the code and how it implements the game:</br>

## Algorithm:
### 1) Setup and Initialization:
The game is configured to use a 4x4 grid, with each tile holding an animal image. These images are loaded from the assets directory, which must contain at least eight distinct .png files for the game to run.</br></br>
The game will start by loading all images and pairing them up randomly across the grid.
### 2) Class Definition (Animal):

The Animal class is responsible for each tile on the board, representing an animal image. Each image appears exactly twice in the game.</br></br>
When an Animal object is created, it picks a random animal image that hasn't yet reached its pair limit.
### 3) Gameplay Functions:
     i) find_index_from_xy(): Converts mouse coordinates to the grid’s row and column indices.</br></br>
     ii) draw_winner_screen(): Displays the winning message, comparing player and AI times.</br></br>
     iii)ai_turn(): Randomly selects two tiles for the AI’s turn.</br></br>
     iv)display_time(): Shows the elapsed time for the player.</br>
### 4) Game Loop (main()):

i)The main game loop handles player and AI turns:</br></br>
     a) Player Turn: The player clicks on two tiles to reveal them. If they match, they stay revealed; otherwise, they are hidden after a short pause.</br></br>
     b) AI Turn: The AI picks two random tiles and checks for a match.

</br> ii)This continues until all pairs are found, and the game ends, showing the total times for both the player and the AI.</br>
### 5) Winner Determination and Time Plotting:

After all pairs are matched, the function draw_winner_screen() determines the winner based on who finished in less time.</br>
The plot_time_comparison() function displays a bar chart comparing the player's and AI's times.</br>
### 6) Menu Screen:
draw_menu_screen(): A simple menu function to display a selection for the game's level, though only one level (4x4) is currently implemented.</br>
## Running the Game:
To run the game, ensure:
</br></br>
     a)The assets directory exists with at least eight .png images.</br>
     b)Pygame and Matplotlib libraries are installed.
## Program:
```
import pygame
import random
import os
from time import sleep, time
import matplotlib.pyplot as plt

# Game configuration
IMAGE_SIZE = 128
MARGIN = 8
ASSET_DIR = 'assets'
SCREEN_SIZE_LEVEL_1 = 512  # 4x4
NUM_TILES_SIDE_LEVEL_1 = 4
NUM_TILES_TOTAL_LEVEL_1 = 16

# Load asset files
try:
    ASSET_FILES = [x for x in os.listdir(ASSET_DIR) if x[-3:].lower() == 'png']
    assert len(ASSET_FILES) >= 8  # Ensure enough animal images for pairs
except FileNotFoundError:
    print(f"Error: The directory '{ASSET_DIR}' was not found. Please create this directory and add images.")
    exit()

# Class for Animals
class Animal:
    animals_count = dict((a, 0) for a in ASSET_FILES)

    @classmethod
    def available_animals(cls):
        return [animal for animal, count in cls.animals_count.items() if count < 2]

    def __init__(self, index, num_tiles_side):
        available = Animal.available_animals()
        if not available:
            raise ValueError("No available animals to choose from.")

        self.index = index
        self.name = random.choice(available)
        self.image_path = os.path.join(ASSET_DIR, self.name)
        self.row = index // num_tiles_side
        self.col = index % num_tiles_side
        self.skip = False
        self.image = pygame.image.load(self.image_path)
        self.image = pygame.transform.scale(self.image, (IMAGE_SIZE - 2 * MARGIN, IMAGE_SIZE - 2 * MARGIN))
        self.box = self.image.copy()
        self.box.fill((200, 200, 200))
        Animal.animals_count[self.name] += 1

def find_index_from_xy(x, y, num_tiles_side):
    row = y // IMAGE_SIZE
    col = x // IMAGE_SIZE
    index = row * num_tiles_side + col
    return row, col, index

def draw_winner_screen(screen, player_time, ai_time):
    font = pygame.font.SysFont(None, 55)
    if player_time < ai_time:
        
        text = font.render(f"You Win! {player_time:.2f}s", True, (0, 255, 0))
    elif ai_time < player_time:
        
        text = font.render(f"AI Wins! {ai_time:.2f}s", True, (255, 0, 0))
    else:
        
        text = font.render("It's a Tie!", True, (0, 0, 255))
        
    screen.blit(text, (screen.get_width() // 2 - text.get_width() // 2, screen.get_height() // 2 - text.get_height() // 2))
    pygame.display.flip()
    sleep(3)  # Show the winner screen for 3 seconds

def ai_turn(tiles, revealed_tiles):
    # AI selects two random tiles from the unrevealed tiles
    available_indices = [i for i in range(len(tiles)) if i not in revealed_tiles]
    if len(available_indices) < 2:
        return []
    
    idx1 = random.choice(available_indices)
    available_indices.remove(idx1)
    idx2 = random.choice(available_indices)

    return [idx1, idx2]

def display_time(screen, player_time):
    font = pygame.font.SysFont(None, 35)
    time_text = font.render(f"Time: {player_time:.2f} s", True, (0, 0, 0))
    screen.blit(time_text, (10, 10))

def main():
    # Fixed grid size for Level 1
    screen_size = SCREEN_SIZE_LEVEL_1
    num_tiles_side = NUM_TILES_SIDE_LEVEL_1
    num_tiles_total = NUM_TILES_TOTAL_LEVEL_1

    pygame.init()
    pygame.display.set_caption('Memory Game')
    screen = pygame.display.set_mode((screen_size, screen_size))
    matched = pygame.image.load('other_assets/matched.png')
    
    running = True
    tiles = [Animal(i, num_tiles_side) for i in range(num_tiles_total)]
    current_images_displayed = []
    
    player_turn = True
    player_time_start = time()  # Start the player timer
    player_time_paused_duration = 0  # Track how much time is paused
    player_time_pause_start = None  # Start tracking when time is paused
    player_time_end = None
    ai_time_start = None
    ai_time_end = None

    total_skipped = 0

    while running:
        current_events = pygame.event.get()

        for e in current_events:
            if e.type == pygame.QUIT:
                running = False

            if e.type == pygame.KEYDOWN:
                if e.key == pygame.K_ESCAPE:
                    running = False

            if e.type == pygame.MOUSEBUTTONDOWN and player_turn:
                # Resume the player's timer when the player clicks to select the next tile
                if player_time_pause_start:
                    player_time_paused_duration += time() - player_time_pause_start
                    player_time_pause_start = None

                mouse_x, mouse_y = pygame.mouse.get_pos()
                row, col, index = find_index_from_xy(mouse_x, mouse_y, num_tiles_side)
                if index not in current_images_displayed and not tiles[index].skip:
                    if len(current_images_displayed) > 1:
                        current_images_displayed = current_images_displayed[1:] + [index]
                    else:
                        current_images_displayed.append(index)

        # Display animals
        screen.fill((255, 255, 255))

        for i, tile in enumerate(tiles):
            current_image = tile.image if i in current_images_displayed else tile.box
            if not tile.skip:
                screen.blit(current_image, (tile.col * IMAGE_SIZE + MARGIN, tile.row * IMAGE_SIZE + MARGIN))

        if player_turn:
            player_time_current = time() - player_time_start - player_time_paused_duration
            display_time(screen, player_time_current)  # Display the player's current time

        pygame.display.flip()

        # Check for matches
        if len(current_images_displayed) == 2:
            idx1, idx2 = current_images_displayed
            if tiles[idx1].name == tiles[idx2].name:
                tiles[idx1].skip = True
                tiles[idx2].skip = True
                # Display matched message
                sleep(0.2)
                screen.blit(matched, (0, 0))
                pygame.display.flip()
                sleep(0.5)

                current_images_displayed = []
            else:
                # Pause the player's timer when the player selects two wrong cards
                player_time_pause_start = time()

                # Display the second card for 1 second before flipping back
                sleep(1)
                current_images_displayed = []

            player_turn = not player_turn  # Switch turns

            # If all tiles are matched, end the game
            total_skipped = sum(1 for tile in tiles if tile.skip)
            if total_skipped == len(tiles):
                running = False
                player_time_end = time()  # Record player time
                if player_turn:
                    player_time_total = player_time_end - player_time_start - player_time_paused_duration
                else:
                    ai_time_end = time()
                    ai_time_total = ai_time_end - ai_time_start
                break

            # AI's turn
            if not player_turn:
                if ai_time_start is None:
                    ai_time_start = time()  # Start AI timer

                sleep(1)  # Give time for the player to see the result
                ai_choices = ai_turn(tiles, [i for i in current_images_displayed if not tiles[i].skip])
                for choice in ai_choices:
                    current_images_displayed.append(choice)
                pygame.display.flip()
                sleep(1)  # Allow AI's selection to be visible

                # Check for matches
                if len(current_images_displayed) == 2:
                    idx1, idx2 = current_images_displayed
                    if tiles[idx1].name == tiles[idx2].name:
                        tiles[idx1].skip = True
                        tiles[idx2].skip = True

                current_images_displayed = []
                player_turn = True  # Switch back to player's turn

                if sum(1 for tile in tiles if tile.skip) == len(tiles):
                    running = False  # End the game when all tiles are matched
                    ai_time_end = time()  # Record AI time

    # Determine times for comparison
    player_time_total = player_time_end - player_time_start - player_time_paused_duration if player_time_end else 0
    ai_time_total = ai_time_end - ai_time_start if ai_time_end else 0
    print()
    print()
    # Show the winner screen
    draw_winner_screen(screen, player_time_total, ai_time_total)

    # Plot the comparison chart
    plot_time_comparison(player_time_total, ai_time_total)

def plot_time_comparison(player_time, ai_time):
    # Create a bar chart comparing the player's time and the AI's time
    labels = ['Player', 'AI']
    times = [player_time, ai_time]

    plt.bar(labels, times, color=['blue', 'red'])
    plt.ylabel('Time (seconds)')
    plt.title('Player vs AI Time Comparison')
    plt.show()

def draw_menu_screen():
    pygame.init()
    screen = pygame.display.set_mode((SCREEN_SIZE_LEVEL_1, SCREEN_SIZE_LEVEL_1))
    pygame.display.set_caption('Select Level')
    font = pygame.font.SysFont(None, 55)

    while True:
        screen.fill((255, 255, 255))
        level_1_text = font.render("Press 1 for Level 1 (4x4)", True, (0, 0, 0))
        screen.blit(level_1_text, (screen.get_width() // 2 - level_1_text.get_width() // 2, 200))

        pygame.display.flip()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_1:
                    pygame.quit()
                    return 1  # Only Level 1

if __name__ == '__main__':
    level = draw_menu_screen()  # Get the level from the menu
    try:
        main()
    except Exception as e:
        print(f"An error occurred: {e}")
    finally:
        pygame.quit()

```
## Output:
![Screenshot 2024-10-04 102507](https://github.com/user-attachments/assets/9330d5e6-aba4-4ad0-abc1-3f73898a87cd)
![Screenshot 2024-10-04 102352](https://github.com/user-attachments/assets/46477262-31cd-4323-913d-0c4d08039b5f)

## Result:
This program is an excellent example of using Python for simple graphical games. It combines logic, basic AI, and time tracking to create an engaging memory game.
