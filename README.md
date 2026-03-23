import pygame, sys, random

# Initialize Pygame and the Mixer
pygame.init()
pygame.mixer.init() 
clock = pygame.time.Clock()

# --- LOAD SOUNDS ---
try:
    hit_sound = pygame.mixer.Sound(r"C:\Users\saleh\OneDrive\Desktop\Python\Ball hiting.mp3")
    score_sound = pygame.mixer.Sound(r"C:\Users\saleh\OneDrive\Desktop\Python\scoring1.mp3")
except:
    print("Sound files not found! Please check paths.")
    hit_sound = None
    score_sound = None

# Screen Setup
screen_width = 1280
screen_height = 960
screen = pygame.display.set_mode((screen_width, screen_height))
pygame.display.set_caption("Ping Pong")

# Colors & Fonts
white = (255, 255, 255)
black = (28, 28, 28)
title_font = pygame.font.Font(None, 80)
huge_font = pygame.font.Font(None, 200)
basic_font = pygame.font.Font(None, 40)
button_font = pygame.font.Font(None, 50)

# Game Objects
ball = pygame.Rect(screen_width/2 - 15, screen_height/2 - 15, 30, 30)
left_paddle = pygame.Rect(15, screen_height/2 - 70, 15, 140)
right_paddle = pygame.Rect(screen_width - 30, screen_height/2 - 70, 15, 140)

# Variables
ball_speed_x, ball_speed_y = 0, 0
left_score, right_score = 0, 0
game_state = "menu"
game_mode = None
countdown_value = 3
last_count_time = 0

# --- UI HELPERS ---
def draw_text(text, font, color, x, y, center=False):
    surf = font.render(text, True, color)
    rect = surf.get_rect(topleft=(x, y))
    if center: rect.centerx = x
    screen.blit(surf, rect)

def draw_button(text, x, y, w, h):
    mouse = pygame.mouse.get_pos()
    rect = pygame.Rect(x, y, w, h)
    color = (50, 50, 50) if rect.collidepoint(mouse) else black
    pygame.draw.rect(screen, color, rect)
    label = button_font.render(text, True, white)
    screen.blit(label, label.get_rect(center=rect.center))
    return rect

def start_countdown():
    global game_state, countdown_value, last_count_time, ball_speed_x, ball_speed_y
    game_state = "countdown"
    countdown_value = 3
    last_count_time = pygame.time.get_ticks() 
    ball.center = (screen_width/2, screen_height/2)
    ball_speed_x, ball_speed_y = 0, 0

def throw_ball():
    global ball_speed_x, ball_speed_y, game_state
    ball_speed_x = 7 * random.choice((1, -1))
    ball_speed_y = 7 * random.choice((1, -1))
    game_state = "game"

# --- MAIN LOOP ---
while True:
    current_time = pygame.time.get_ticks()
    mouse_pos = pygame.mouse.get_pos()

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit(); sys.exit()
        
        if event.type == pygame.MOUSEBUTTONDOWN:
            if game_state == "menu":
                if b1.collidepoint(mouse_pos): game_state = "mode"
                if b2.collidepoint(mouse_pos): pygame.quit(); sys.exit()
            elif game_state == "mode":
                if b_pvp.collidepoint(mouse_pos): game_mode = "pvp"; game_state = "ins_pvp"
                if b_ai.collidepoint(mouse_pos): game_mode = "ai"; game_state = "ins_ai"
            elif game_state == "ins_pvp" or game_state == "ins_ai":
                if b_play.collidepoint(mouse_pos): start_countdown()
            elif game_state in ["game", "countdown"]:
                if b_quit_game.collidepoint(mouse_pos): game_state = "result"
            elif game_state == "result":
                if b_again.collidepoint(mouse_pos): left_score, right_score = 0, 0; game_state = "menu"
                if b_quit_res.collidepoint(mouse_pos): pygame.quit(); sys.exit()

    # --- STATE RENDERING ---
    screen.fill(white)

    if game_state == "menu":
        draw_text("PingPong Game", title_font, black, screen_width/2, 200, True)
        b1 = draw_button("Continue", 540, 350, 200, 70)
        b2 = draw_button("Quit", 540, 450, 200, 70)

    elif game_state == "mode":
        draw_text("Select game mode", title_font, black, screen_width/2, 150, True)
        b_pvp = draw_button("1 Vs 1", 350, 400, 240, 120)
        b_ai = draw_button("1 Vs Robot", 690, 400, 240, 120)

    elif game_state == "ins_pvp":
        draw_text("1 Vs 1", title_font, black, screen_width/2, 100, True)
        draw_text("Play against a friend on the same device.", basic_font, black, screen_width/2, 220, True)
        draw_text("Player 1: Press 'W' and 'S' to play.", basic_font, black, screen_width/2, 300, True)
        draw_text("Player 2: Press UP / DOWN arrow keys to play.", basic_font, black, screen_width/2, 340, True)
        b_play = draw_button("Play", 540, 550, 200, 70)

    elif game_state == "ins_ai":
        draw_text("1 Vs Robot", title_font, black, screen_width/2, 100, True)
        draw_text("Play against an AI opponent. Great for practice.", basic_font, black, screen_width/2, 250, True)
        draw_text("Robot is on the LEFT. You are on the RIGHT.", basic_font, black, screen_width/2, 300, True)
        draw_text("Use UP/Down key to play.", basic_font, black, screen_width/2, 350, True)
        b_play = draw_button("Play", 540, 550, 200, 70)

    elif game_state in ["game", "countdown"]:
        # Logic
        if game_state == "countdown":
            if current_time - last_count_time > 1000:
                countdown_value -= 1
                last_count_time = current_time
                if countdown_value <= 0: throw_ball()
        
        if game_state == "game":
            ball.x += ball_speed_x
            ball.y += ball_speed_y
            if ball.top <= 0 or ball.bottom >= screen_height:
                ball_speed_y *= -1
                if hit_sound: hit_sound.play()
            if ball.colliderect(left_paddle) or ball.colliderect(right_paddle):
                ball_speed_x *= -1
                if hit_sound: hit_sound.play()
            if ball.left <= 0:
                right_score += 1
                if score_sound: score_sound.play()
                start_countdown()
            if ball.right >= screen_width:
                left_score += 1
                if score_sound: score_sound.play()
                start_countdown()

        # Input
        keys = pygame.key.get_pressed()
        if game_mode == "pvp":
            if keys[pygame.K_w]: left_paddle.y -= 8
            if keys[pygame.K_s]: left_paddle.y += 8
        else:
            if left_paddle.centery < ball.centery: left_paddle.y += 6
            elif left_paddle.centery > ball.centery: left_paddle.y -= 6
        if keys[pygame.K_UP]: right_paddle.y -= 8
        if keys[pygame.K_DOWN]: right_paddle.y += 8

        left_paddle.clamp_ip(screen.get_rect())
        right_paddle.clamp_ip(screen.get_rect())

        # Draw
        pygame.draw.rect(screen, black, left_paddle)
        pygame.draw.rect(screen, black, right_paddle)
        pygame.draw.ellipse(screen, black, ball)
        pygame.draw.aaline(screen, black, (screen_width/2, 0), (screen_width/2, screen_height))
        draw_text(str(left_score), title_font, black, screen_width/2 - 80, screen_height/2 - 30)
        draw_text(str(right_score), title_font, black, screen_width/2 + 40, screen_height/2 - 30)
        
        if game_state == "countdown":
            draw_text(str(countdown_value), huge_font, black, screen_width/2, screen_height/2 - 200, True)
        
        b_quit_game = draw_button("Quit", 10, 10, 120, 50)
        if left_score >= 5 or right_score >= 5: game_state = "result"

    elif game_state == "result":
        draw_text("Game Over", title_font, black, screen_width/2, 200, True)
        draw_text(f"{left_score} : {right_score}", title_font, black, screen_width/2, 300, True)
        b_again = draw_button("Play Again", 520, 450, 240, 70)
        b_quit_res = draw_button("Quit", 520, 550, 240, 70)

    pygame.display.flip()
    clock.tick(60)
