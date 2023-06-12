# Programiranje-2-Connect-4
Projekat za P2 FPMOZ
import pygame
import numpy as np
import sys
import math
import time



WHITE = (255, 255, 255)
RED = (255, 51, 51)
BLUE=(204,255,255)
BLUE2=(36,48,98)
YELLOW=(255,255,51)

pygame.init()


board_cols=7
board_rows=6

def create_board():
	board = np.zeros((board_rows,board_cols))
	return board

DARK_BLUE = (19, 41, 61)
DARK_YELLOW = (200, 200, 0)   
DARK_RED = (200, 0, 0)   
SHADOW_OFFSET = 5 
SHADOW_SIZE = 1

def draw_board(board):
    for c in range(board_cols):
        for r in range(board_rows):
            pygame.draw.rect(screen, BLUE2, (c*SQUARESIZE, r*SQUARESIZE+SQUARESIZE, SQUARESIZE, SQUARESIZE))
            pygame.draw.circle(screen, BLUE, (int(c*SQUARESIZE+SQUARESIZE/2), int(r*SQUARESIZE+SQUARESIZE+SQUARESIZE/2)), RADIUS)

    for c in range(board_cols):
        for r in range(board_rows):
            if board[r][c] == 1:
                pygame.draw.circle(screen, DARK_RED, (int(c*SQUARESIZE+SQUARESIZE/2), height-int(r*SQUARESIZE+SQUARESIZE/2)), RADIUS)
                pygame.draw.circle(screen, RED, (int(c*SQUARESIZE+SQUARESIZE/2) + SHADOW_OFFSET, height-int(r*SQUARESIZE+SQUARESIZE/2) + SHADOW_OFFSET), RADIUS + SHADOW_SIZE)
            elif board[r][c] == 2:
                pygame.draw.circle(screen, DARK_YELLOW, (int(c*SQUARESIZE+SQUARESIZE/2), height-int(r*SQUARESIZE+SQUARESIZE/2)), RADIUS)
                pygame.draw.circle(screen, YELLOW, (int(c*SQUARESIZE+SQUARESIZE/2) + SHADOW_OFFSET, height-int(r*SQUARESIZE+SQUARESIZE/2) + SHADOW_OFFSET), RADIUS + SHADOW_SIZE)
    pygame.display.update()
    
board = create_board()



SQUARESIZE = 100

width = board_cols * SQUARESIZE
height = (board_rows+1) * SQUARESIZE

size = (width, height)

RADIUS = int(SQUARESIZE/2 - 5)

screen = pygame.display.set_mode(size)
draw_board(board)
pygame.display.update()

myfont = pygame.font.SysFont("monospace", 75)

# Function to check if a player has won
def check_win(board, player):
    # Check horizontally
    for row in range(board_rows):
        for col in range(board_cols - 3):
            if board[row][col] == board[row][col + 1] == board[row][col + 2] == board[row][col + 3] == player:
                return True

    # Check vertically
    for row in range(board_rows - 3):
        for col in range(board_cols):
            if board[row][col] == board[row + 1][col] == board[row + 2][col] == board[row + 3][col] == player:
                return True

    # Check diagonally (top-left to bottom-right)
    for row in range(board_rows - 3):
        for col in range(board_cols - 3):
            if board[row][col] == board[row + 1][col + 1] == board[row + 2][col + 2] == board[row + 3][col + 3] == player:
                return True

    # Check diagonally (bottom-left to top-right)
    for row in range(3, board_rows):
        for col in range(board_cols - 3):
            if board[row][col] == board[row - 1][col + 1] == board[row - 2][col + 2] == board[row - 3][col + 3] == player:
                return True

    return False

# Function to check if the game is a draw
def check_draw(board):
    for row in board:
        if 0 in row:
            return False
    return True





def calculate_score(lines, opponent):
    score = 0
    if lines.count(opponent) == 3 and lines.count(0) == 1:
        score -= 4
    if lines.count(opponent) == 2 and lines.count(0) == 2:
        score -= 2
    return score
'''
def calculate_score(lines, player, opponent):
    score = 0
    if lines.count(player) == 3 and lines.count(0) == 1:
        score += 5
    elif lines.count(player) == 2 and lines.count(0) == 2:
        score += 2
    elif lines.count(opponent) == 3 and lines.count(0) == 1:
        score -= 4
    elif lines.count(opponent) == 2 and lines.count(0) == 2:
        score -= 2
    
    return score
'''#pocetni kod,ai ne sprjecava pobjedu igraca vec daje prednost svojoj pobjedi,

def evaluate_board(board, player):
    opponent = 1 if player == 2 else 2
    score = 0

    if check_win(board, opponent):
        return -100

    if check_win(board, player):
        return 100

    for row in range(6):
        for col in range(4):
            lines = [board[row][col], board[row][col + 1], board[row][col + 2], board[row][col + 3]]
            score += calculate_score(lines, opponent)
    for row in range(3):
        for col in range(7):
            lines = [board[row][col], board[row + 1][col], board[row + 2][col], board[row + 3][col]]
            score += calculate_score(lines, opponent)

    for row in range(3):
        for col in range(4):
            lines = [board[row][col], board[row + 1][col + 1], board[row + 2][col + 2], board[row + 3][col + 3]]
            score += calculate_score(lines, opponent)

    for row in range(3, 6):
        for col in range(4):
            lines = [board[row][col], board[row - 1][col + 1], board[row - 2][col + 2], board[row - 3][col + 3]]
            score += calculate_score(lines, opponent)

    return score


def make_move(board, col, player):
    if 0 in board[:, col]:
        for row in range(board_rows):
            if board[row, col] == 0:
                board[row, col] = player
                if row == board_rows - 1:
                    return True
                break
    else:
        return False

def make_ai_move(board, player):
    available_cols = [col for col in range(7) if np.any(board[:, col] == 0)]
    best_score = float('-inf')
    best_move = None
    opponent = 1 if player == 2 else 2

    for col in available_cols:
        temp_board = np.copy(board)
        make_move(temp_board, col, player)

        if check_win(temp_board, opponent):
            best_move = col
            break

        score = evaluate_board(temp_board, player)
        if score > best_score:
            best_score = score
            best_move = col

    make_move(board, best_move, player)
    
def is_column_full(board, col):
    for row in range(board_rows):
        if board[row][col] == 0:
            return False
    return True

def show_message(message_text):
    message = myfont.render(message_text, True, BLUE2)
    message_rect = message.get_rect(center=(screen_width // 2,50))

    screen.blit(message, message_rect)
    pygame.display.update()
    
    
def play_vs_ai():
    player1 = 1
    player2 = 2
    posx = 0

    screen.fill(BLUE)
    draw_board(board)
    pygame.display.update()
    game_over=False

    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.MOUSEMOTION:
                posx = event.pos[0]
                pygame.draw.rect(screen, BLUE, (0, 0, width, SQUARESIZE))
                if player1 == 1:
                    pygame.draw.circle(screen, DARK_RED, (posx, int(SQUARESIZE / 2)), RADIUS)
                    pygame.draw.circle(screen, RED, (posx + SHADOW_OFFSET, int(SQUARESIZE / 2) + SHADOW_OFFSET), RADIUS + SHADOW_SIZE)
                else:
                    pygame.draw.circle(screen, DARK_YELLOW, (posx, int(SQUARESIZE / 2)), RADIUS)
                    pygame.draw.circle(screen, YELLOW, (posx + SHADOW_OFFSET, int(SQUARESIZE / 2) + SHADOW_OFFSET), RADIUS + SHADOW_SIZE)
                draw_board(board)
                pygame.display.update()

            if event.type == pygame.MOUSEBUTTONDOWN:
                pygame.draw.rect(screen, BLUE, (0, 0, width, SQUARESIZE))
                posx = event.pos[0]
                col = int(math.floor(posx / SQUARESIZE))

                if is_column_full(board, col):
                    show_message("Column is full!")
                    pygame.display.update()
                    continue

                if player1==1:
                    make_move(board,col,player1)
                else:
                    make_ai_move(board,player1)
                if player1==1:
                    if check_win(board, player1):
                        draw_board(board)
                        show_message("Player 1 wins!")
                        pygame.display.update()
                        pygame.time.wait(4000)
                        game_over=True
                if player1==2:
                    if check_win(board, player1):
                        draw_board(board)
                        show_message("AI wins!")
                        pygame.display.update()
                        pygame.time.wait(4000)
                        game_over=True
                
                draw_board(board)
                pygame.display.update()
                
                if check_draw(board):
                    show_message("It's a draw!")
                    pygame.display.update()
                    pygame.time.wait(4000)
                    game_over=True
                    
                if game_over==True:
                        play_again()
        
                player1, player2 = player2, player1
                

def play_vs_player():
    player1 = 1
    player2 = 2
    posx = 0

    screen.fill(BLUE)
    draw_board(board)
    pygame.display.update()
    game_over=False

    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.MOUSEMOTION:
                posx = event.pos[0]
                pygame.draw.rect(screen, BLUE, (0, 0, width, SQUARESIZE))
                if player1 == 1:
                    pygame.draw.circle(screen, DARK_RED, (posx, int(SQUARESIZE / 2)), RADIUS)
                    pygame.draw.circle(screen, RED, (posx + SHADOW_OFFSET, int(SQUARESIZE / 2) + SHADOW_OFFSET), RADIUS + SHADOW_SIZE)
                else:
                    pygame.draw.circle(screen, DARK_YELLOW, (posx, int(SQUARESIZE / 2)), RADIUS)
                    pygame.draw.circle(screen, YELLOW, (posx + SHADOW_OFFSET, int(SQUARESIZE / 2) + SHADOW_OFFSET), RADIUS + SHADOW_SIZE)
                draw_board(board)
                pygame.display.update()

            if event.type == pygame.MOUSEBUTTONDOWN:
                pygame.draw.rect(screen, BLUE, (0, 0, width, SQUARESIZE))
                posx = event.pos[0]
                col = int(math.floor(posx / SQUARESIZE))

                if is_column_full(board, col):
                    show_message("Column is full!")
                    pygame.display.update()
                    continue

                
                make_move(board, col, player1)
                
                if player1==1:
                    if check_win(board, player1):
                        draw_board(board)
                        show_message("Red wins!")
                        pygame.display.update()
                        pygame.time.wait(4000)
                        game_over=True
                        
                if player1==2:
                    if check_win(board, player1):
                        draw_board(board)
                        show_message("Yellow wins!")
                        pygame.display.update()
                        pygame.time.wait(4000)
                        game_over=True
                        
                
                
                draw_board(board)
                pygame.display.update()
                
            
                if check_draw(board):
                    show_message("It's a draw!")
                    pygame.display.update()
                    pygame.time.wait(3000)
                    game_over=True

                if game_over==True:
                        play_again()

                
        
                player1,player2=player2,player1
                

screen_width=700
screen_height=700
button_width = 350
button_height = 50
button1_pos = (screen_width // 2 - button_width // 2, screen_height // 2 - button_height)
button2_pos = (screen_width // 2 - button_width // 2, screen_height // 2 + button_height)

def clear_board(board):
    for row in range(len(board)):
        for col in range(len(board[row])):
            board[row][col] = 0
            
def play_again():
    clear_board(board)
    draw_start_screen()
    


def draw_start_screen():
    screen = pygame.display.set_mode((screen_width, screen_height))
    screen.fill(BLUE)

    font = pygame.font.Font(None, 50)
    title_text = font.render("Choose Game Mode", True, BLUE2)
    title_text_rect = title_text.get_rect(center=(screen_width // 2, screen_height // 2 - 150))
    screen.blit(title_text, title_text_rect)

    pygame.draw.rect(screen, BLUE2, (button1_pos[0], button1_pos[1], button_width, button_height))
    font = pygame.font.Font(None, 24)
    option1_text = font.render("PLAY AGAINST AI", True, WHITE)
    option1_text_rect = option1_text.get_rect(center=(screen_width // 2, screen_height // 2 - 25))
    screen.blit(option1_text, option1_text_rect)

    pygame.draw.rect(screen, BLUE2, (button2_pos[0], button2_pos[1], button_width, button_height))
    option2_text = font.render("PLAY AGAINST PLAYER", True, WHITE)
    option2_text_rect = option2_text.get_rect(center=(screen_width // 2, screen_height // 2 + 75))
    screen.blit(option2_text, option2_text_rect)

    pygame.display.flip()

    running = True

    while running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

            elif event.type == pygame.MOUSEBUTTONDOWN:
                mouse_pos = pygame.mouse.get_pos()

                if button1_pos[0] <= mouse_pos[0] <= button1_pos[0] + button_width and \
                        button1_pos[1] <= mouse_pos[1] <= button1_pos[1] + button_height:
                    play_vs_ai()
                    running = False

                elif button2_pos[0] <= mouse_pos[0] <= button2_pos[0] + button_width and \
                        button2_pos[1] <= mouse_pos[1] <= button2_pos[1] + button_height:
                    play_vs_player()
                    running = False

draw_start_screen()
                    

            



pygame.quit()
