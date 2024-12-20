```python
import random  # for generating random numbers
import sys  # To Exit the game
import pygame
from pygame.locals import *  # Basic Pygame Imports

# Constants
FPS = 32
SCREENWIDTH = 750
SCREENHEIGHT = 420
GROUNDY = SCREENHEIGHT
PLAYER = 'resources/SPRITES/bird.png'
BACKGROUND = 'resources/SPRITES/SF_small.jpeg'
PIPE = 'resources/SPRITES/pipe.png'

# Initialize Pygame
pygame.init()
FPSCLOCK = pygame.time.Clock()

# Set up the screen
SCREEN = pygame.display.set_mode((SCREENWIDTH, SCREENHEIGHT))

# Load game resources (images, sounds)
GAME_SPRITES['player'] = pygame.image.load(PLAYER).convert_alpha()
GAME_SPRITES['background'] = pygame.image.load(BACKGROUND).convert()
GAME_SPRITES['message'] = pygame.image.load('resources/SPRITES/message.png').convert_alpha()

# Main function to show the welcome screen
def welcomeScreen():
    playerx = int(SCREENWIDTH / 5)
    playery = int(SCREENHEIGHT - GAME_SPRITES['player'].get_height()) / 2
    messagex = int(SCREENWIDTH - GAME_SPRITES['message'].get_width()) / 2
    messagey = int(SCREENHEIGHT * 0.13)
    basex = 0

    # Drawing Rectangle for play button
    playbutton = pygame.Rect(325, 222, 68, 65)

    while True:
        for event in pygame.event.get():
            # If user clicks on Cross or presses the Escape button, close the game
            if event.type == QUIT or (event.type == KEYDOWN and event.key == K_ESCAPE):
                pygame.quit()
                sys.exit()

            # If user presses Space or Up arrow, start the main game
            elif event.type == KEYDOWN and (event.key == K_SPACE or event.key == K_UP):
                return

            pygame.mouse.set_cursor(pygame.SYSTEM_CURSOR_ARROW)
            
            # Change cursor to hand if it's over the play button
            if pygame.mouse.get_pos()[0] > playbutton[0] and pygame.mouse.get_pos()[0] < playbutton[0] + playbutton[2]:
                if pygame.mouse.get_pos()[1] > playbutton[1] and pygame.mouse.get_pos()[1] < playbutton[1] + playbutton[3]:
                    pygame.mouse.set_cursor(pygame.SYSTEM_CURSOR_HAND)

            # Check if mouse is collided with the play button
            if playbutton.collidepoint(pygame.mouse.get_pos()):
                # Check if mouse has been clicked
                if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                    mainGame()

            else:
                SCREEN.blit(GAME_SPRITES['background'], (0, 0))
                SCREEN.blit(GAME_SPRITES['player'], (playerx, playery))
                SCREEN.blit(GAME_SPRITES['message'], (messagex, messagey))
                
                # Adding Intro Music
                pygame.mixer.music.load('resources/AUDIO/INTROMUSIC.mp3')
                pygame.mixer.music.play()
                
                pygame.display.update()
                FPSCLOCK.tick(FPS)

def mainGame():

    # ADDING THE BACKGROUND MUSIC
    pygame.mixer.music.stop()
    pygame.mixer.music.load('resources/AUDIO/BGMUSIC.mp3')
    pygame.mixer.music.play()
    
    score = 0
    playerx = int(SCREENWIDTH/5)
    playery = int(SCREENHEIGHT/2)
    basex = 0

    # Creating upper and Lower Pipes for the game
    newPipe1 = getRandomPipe()
    newPipe2 = getRandomPipe()

    # Upper pipe List
    upperPipes = [
        {'x': SCREENWIDTH + 200, 'y': newPipe1[0]['y']},
        {'x': SCREENWIDTH + 200 + (SCREENWIDTH/2), 'y': newPipe2[0]['y']}
    ]

    # Lists of Lower Pipe
    lowerPipes = [
        {'x': SCREENWIDTH + 200, 'y': newPipe1[1]['y']},
        {'x': SCREENWIDTH + 200 + (SCREENWIDTH/2), 'y': newPipe2[1]['y']}
    ]

    pipeVelX = -4
    playerVelY = -9
    playerMaxVelY = 10
    playerMinVelY = -8
    playerAccY = 1

    playerFlapAccv = -8  # velocity while flapping
    playerFlapped = False  # It is true only when the bird is flapping

    while True:

        for event in pygame.event.get():
            if event.type == QUIT or (event.type == KEYDOWN and event.key == K_ESCAPE):
                pygame.quit()
                sys.exit()

            if event.type == KEYDOWN and (event.key == K_SPACE or event.key == K_UP):
                if playery > 0:
                    playerVelY = playerFlapAccv
                    playerFlapped = True
                    GAME_SOUNDS['wing'].play()

        crashTest = isCollide(playerx, playery, upperPipes, lowerPipes)  # This function will return true if the player is crashed
        if crashTest:
            return

        # Check for score
        playerMidPos = playerx + GAME_SPRITES['player'].get_width()/2
        for pipe in upperPipes:
            pipeMidPos = pipe['x'] + GAME_SPRITES['pipe'][0].get_width()/2
            if pipeMidPos <= playerMidPos < pipeMidPos + 4:
                score += 1
                print(f"Your score is {score}")
                GAME_SOUNDS['point'].play()

        if playerVelY < playerMaxVelY and not playerFlapped:
            playerVelY += playerAccY

        if playerFlapped:
            playerFlapped = False

        playerHeight = GAME_SPRITES['player'].get_height()
        playery = playery + min(playerVelY, GROUNDY - playery - playerHeight)

        # Move pipes to the left
        for upperPipe, lowerPipe in zip(upperPipes, lowerPipes):
            upperPipe['x'] += pipeVelX
            lowerPipe['x'] += pipeVelX

        # Add a new pipe when the first is about to cross the leftmost part of the screen
        if 0 < upperPipes[0]['x'] < 5:
            newpipe = getRandomPipe()
            upperPipes.append(newpipe[0])
            lowerPipes.append(newpipe[1])

        # If the pipe is out of the screen, remove it
        if upperPipes[0]['x'] < -GAME_SPRITES['pipe'][0].get_width():
            upperPipes.pop(0)
            lowerPipes.pop(0)

        # Let's blit our sprites now
        SCREEN.blit(GAME_SPRITES['background'], (0, 0))
        for upperPipe, lowerPipe in zip(upperPipes, lowerPipes):
            SCREEN.blit(GAME_SPRITES['pipe'][0], (upperPipe['x'], upperPipe['y']))
            SCREEN.blit(GAME_SPRITES['pipe'][1], (lowerPipe['x'], lowerPipe['y']))

        SCREEN.blit(GAME_SPRITES['player'], (playerx, playery))
        myDigits = [int(x) for x in list(str(score))]
        width = 0
        for digit in myDigits:
            width += GAME_SPRITES['numbers'][digit].get_width()
        Xoffset = (SCREENWIDTH - width) / 2

        for digit in myDigits:
            SCREEN.blit(GAME_SPRITES['numbers'][digit], (Xoffset, SCREENHEIGHT * 0.12))
            Xoffset += GAME_SPRITES['numbers'][digit].get_width()
        pygame.display.update()
        FPSCLOCK.tick(FPS)

def isCollide(playerx, playery, upperPipes, lowerPipes):
    # Check if the player collides with the ground or goes above the screen
    if playery > GROUNDY - 25 or playery < 0:
        GAME_SOUNDS['hit'].play()
        pygame.mixer.music.stop()
        gameOver()

    # Check if the player collides with upper pipes
    for pipe in upperPipes:
        pipeHeight = GAME_SPRITES['pipe'][0].get_height()
        if (playery < pipeHeight + pipe['y'] and abs(playerx - pipe['x']) < GAME_SPRITES['pipe'][0].get_width() - 20):
            GAME_SOUNDS['hit'].play()
            print(playerx, pipe['x'],)
            pygame.mixer.music.stop()
            gameOver()

    # Check if the player collides with lower pipes
    for pipe in lowerPipes:
        if (playery + GAME_SPRITES['player'].get_height() > pipe['y']) and abs(playerx - pipe['x']) < GAME_SPRITES['pipe'][0].get_width() - 20:
            GAME_SOUNDS['hit'].play()
            pygame.mixer.music.stop()
            gameOver()

    return False


def getRandomPipe():
    """
    Generate positions of the two pipes, one upper pipe and the other lower pipe 
    to blit on the screen.
    """
    pipeHeight = GAME_SPRITES['pipe'][0].get_height()
    offset = SCREENHEIGHT / 3.5
    y2 = offset + random.randrange(0, int(SCREENHEIGHT - 1.2 * offset))
    pipeX = SCREENWIDTH + 10
    y1 = pipeHeight - y2 + offset
    pipe = [
        {'x': pipeX, 'y': -y1},  # Upper Pipes
        {'x': pipeX, 'y': y2}  # Lower Pipes
    ]
    return pipe


def gameOver():
    SCREEN = pygame.display.set_mode((SCREENWIDTH, SCREENHEIGHT))
    pygame.display.set_caption('Flying Over SF')
    GAME_SPRITES['OVER'] = pygame.image.load('resources/SPRITES/gameover.png').convert_alpha()
    GAME_SPRITES['RETRY'] = pygame.image.load('resources/SPRITES/retry.png').convert_alpha()
    GAME_SPRITES['HOME'] = pygame.image.load('resources/SPRITES/Home.png').convert_alpha()
    SCREEN.blit(GAME_SPRITES['background'], (0, 0))
    SCREEN.blit(GAME_SPRITES['OVER'], (225, 0))
    SCREEN.blit(GAME_SPRITES['RETRY'], (255, 220))
    SCREEN.blit(GAME_SPRITES['HOME'], (255, 280))

    pygame.display.update()

# Main game loop
while True:
    for event in pygame.event.get():
        # Check if the user quits the game or presses the ESC key
        if event.type == QUIT or (event.type == KEYDOWN and event.key == K_ESCAPE):
            pygame.quit()
            sys.exit()

        # Check if the user presses the SPACE key to start the game
        if event.type == KEYDOWN and event.key == K_SPACE:
            mainGame()

        # Retry button logic
        pygame.mouse.set_cursor(pygame.SYSTEM_CURSOR_ARROW)
        if 255 < pygame.mouse.get_pos()[0] < 255 + GAME_SPRITES['RETRY'].get_width():
            if 220 < pygame.mouse.get_pos()[1] < 220 + GAME_SPRITES['RETRY'].get_height():
                pygame.mouse.set_cursor(pygame.SYSTEM_CURSOR_HAND)
                if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                    mainGame()

        # Home button logic
        if 255 < pygame.mouse.get_pos()[0] < 255 + GAME_SPRITES['HOME'].get_width():
            if 280 < pygame.mouse.get_pos()[1] < 280 + GAME_SPRITES['HOME'].get_height():
                pygame.mouse.set_cursor(pygame.SYSTEM_CURSOR_HAND)
                if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                    welcomeScreen()


# Entry point for the game
if __name__ == "__main__":
    pygame.init()  # Initializing the Modules of Pygame
    FPSCLOCK = pygame.time.Clock()  # For controlling the FPS
    pygame.display.set_caption('Flying Over SF')  # Setting the Caption of The Game

    # Loading the Sprites
    GAME_SPRITES['numbers'] = (
        pygame.image.load('resources/SPRITES/0.png').convert_alpha(),
        pygame.image.load('resources/SPRITES/1.png').convert_alpha(),
        pygame.image.load('resources/SPRITES/2.png').convert_alpha(),
        pygame.image.load('resources/SPRITES/3.png').convert_alpha(),
        pygame.image.load('resources/SPRITES/4.png').convert_alpha(),
        pygame.image.load('resources/SPRITES/5.png').convert_alpha(),
        pygame.image.load('resources/SPRITES/6.png').convert_alpha(),
        pygame.image.load('resources/SPRITES/7.png').convert_alpha(),
        pygame.image.load('resources/SPRITES/8.png').convert_alpha(),
        pygame.image.load('resources/SPRITES/9.png').convert_alpha(),
    )

    GAME_SPRITES['background'] = pygame.image.load(BACKGROUND).convert_alpha()
    GAME_SPRITES['player'] = pygame.image.load(PLAYER).convert_alpha()
    GAME_SPRITES['message'] = pygame.image.load('resources/SPRITES/message.png').convert_alpha()
    GAME_SPRITES['pipe'] = (
        pygame.transform.rotate(pygame.image.load(PIPE).convert_alpha(), 180),  # Upper Pipes, rotated by 180deg
        pygame.image.load(PIPE).convert_alpha()  # Lower Pipes
    )

    # Game Sounds
    GAME_SOUNDS['die'] = pygame.mixer.Sound('resources/AUDIO/die.wav')
    GAME_SOUNDS['hit'] = pygame.mixer.Sound('resources/AUDIO/hit.wav')
    GAME_SOUNDS['point'] = pygame.mixer.Sound('resources/AUDIO/point.wav')
    GAME_SOUNDS['swoosh'] = pygame.mixer.Sound('resources/AUDIO/swoosh.wav')
    GAME_SOUNDS['wing'] = pygame.mixer.Sound('resources/AUDIO/wing.wav')

    while True:
        welcomeScreen()  # Shows a welcome screen to the user until they start the game
        mainGame()  # This is our main game function
```
