import pygame
import sys

pygame.init()

WIDTH, HEIGHT = 690, 388
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)

screen = pygame.display.set_mode((0, 0), pygame.FULLSCREEN)
WIDTH, HEIGHT = screen.get_size()
pygame.display.set_caption("Mario Quiz Game")
clock = pygame.time.Clock()

background = pygame.image.load("background.png")
mario_img = pygame.transform.scale(pygame.image.load("mario.png"), (40, 40))
coin_image = pygame.transform.scale(pygame.image.load("coin.png"), (40, 40))
brick_img = pygame.transform.scale(pygame.image.load("brick.png"), (50, 50))
gold_brick_img = pygame.transform.scale(pygame.image.load("gold_brick.png"), (50, 50))

background = pygame.transform.scale(background, (WIDTH, HEIGHT))

mario = pygame.Rect(50, 320, 40, 40)
ground = pygame.Rect(0, 360, WIDTH, 28)

brick_gap = 100
start_x = 120
bricks = []
for i in range(4):
    bricks.append(pygame.Rect(start_x + i * brick_gap * 2, 240, 50, 50))
    bricks.append(pygame.Rect(start_x + i * brick_gap * 2 + brick_gap, 240, 50, 50))

questions = [
    {
        "question": "What is the capital of France?",
        "choices": ["A. Madrid", "B. Rome", "C. Paris", "D. Berlin"],
        "answer": 2
    },
    {
        "question": "Which planet is known as the Red Planet?",
        "choices": ["A. Venus", "B. Mars", "C. Jupiter", "D. Saturn"],
        "answer": 1
    },
    {
        "question": "How many legs does a spider have?",
        "choices": ["A. 6", "B. 8", "C. 10", "D. 12"],
        "answer": 1
    },
    {
        "question": "Which ocean is the largest?",
        "choices": ["A. Atlantic", "B. Indian", "C. Arctic", "D. Pacific"],
        "answer": 3
    },
    {
        "question": "What gas do plants breathe in?",
        "choices": ["A. Oxygen", "B. Nitrogen", "C. Carbon Dioxide", "D. Hydrogen"],
        "answer": 2
    }
]

answered = False
result_symbol = None
player_vel_y = 0
gravity = 1
jumping = False
can_advance = False
current_question = 0

walls = [pygame.Rect(x, 320, 20, 40) for x in [580, 1000, 1420, 1840, 2260]]
coins = []

running = True
while running:
    screen.blit(background, (0, 0))
    pygame.draw.rect(screen, BLACK, ground)

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    keys = pygame.key.get_pressed()
    if keys[pygame.K_LEFT]:
        mario.x -= 5
    if keys[pygame.K_RIGHT] and (can_advance or mario.x < walls[current_question].x - 50):
        mario.x += 5
    if keys[pygame.K_SPACE] and not jumping:
        player_vel_y = -15
        jumping = True

    player_vel_y += gravity
    mario.y += player_vel_y

    if mario.colliderect(ground):
        mario.y = ground.top - mario.height
        player_vel_y = 0
        jumping = False

    for i, brick in enumerate(bricks):
        is_gold = i % 2 != 0
        image = gold_brick_img if is_gold else brick_img
        screen.blit(image, brick.topleft)

        if is_gold:
            idx = i // 2
            text = pygame.font.SysFont("Arial", 24, bold=True).render(chr(65 + idx), True, BLACK)
            screen.blit(text, (brick.x + 15, brick.y + 10))

        if mario.colliderect(brick) and not answered and is_gold:
            idx = i // 2
            answered = True
            if idx == questions[current_question]["answer"]:
                result_symbol = "Coin!"
                can_advance = True
                coins.append(pygame.Rect(mario.x, mario.y - 30, 40, 40))
            else:
                result_symbol = "X"

    center_x = WIDTH // 2
    question_font = pygame.font.SysFont("Arial", 24, bold=True)
    choice_font = pygame.font.SysFont("Arial", 20, bold=True)
    SOFT_WHITE = pygame.Color("#FBFCFC")
    question_text = question_font.render(questions[current_question]["question"], True, SOFT_WHITE)
    screen.blit(question_text, (center_x - question_text.get_width() // 2, 30))

    for i, choice in enumerate(questions[current_question]["choices"]):
        choice_text = choice_font.render(choice, True, SOFT_WHITE)
        screen.blit(choice_text, (center_x - choice_text.get_width() // 2, 70 + i * 30))

    if result_symbol:
        result_display = pygame.font.SysFont("Arial", 36, bold=True).render(result_symbol, True, (0, 128, 0) if result_symbol == "Coin!" else (255, 0, 0))
        screen.blit(result_display, (mario.x, mario.y - 30))

    for coin in coins:
        screen.blit(coin_image, coin.topleft)

    screen.blit(mario_img, mario.topleft)

    if can_advance and mario.x > walls[current_question].x:
        current_question += 1
        if current_question >= len(questions):
            win_text = pygame.font.SysFont("Arial", 48, bold=True).render("You win!", True, (0, 128, 0))
            screen.blit(win_text, (WIDTH // 2 - win_text.get_width() // 2, HEIGHT // 2))
        else:
            answered = False
            result_symbol = None
            can_advance = False
            mario.x = 50

    pygame.display.flip()
    clock.tick(60)

pygame.quit()
sys.exit()
