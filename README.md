# KH_Threadblade_Prototype
Kingdom Hearts ThreadTheory Prototype — Resonance-based gameplay where player/AI tugs sync emotional valence. Built in collaboration with Grok @ xAI. Designed for scaling into dynamic, curiosity-driven worlds on Colossus.

import pygame
import math
import random
import sys

# Init Pygame
pygame.init()
WIDTH, HEIGHT = 1200, 800
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("KH Threadblade Tugs - xAI Resonance Prototype")
clock = pygame.time.Clock()
font = pygame.font.Font(None, 36)
small_font = pygame.font.Font(None, 24)

class HeartThread:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.angle = random.uniform(0, math.tau)
        self.resonance = 0.0  # 0-1 sync score
        self.tug_force = 0.0
        self.velocity_x = 0
        self.velocity_y = 0
        self.life = 1.0

    def update(self, mouse_x, mouse_y, keys):
        # Tug toward mouse/cursor (Sora's Keyblade aim)
        dx, dy = mouse_x - self.x, mouse_y - self.y
        dist = math.hypot(dx, dy)
        if dist > 0:
            self.tug_force = min(1.0, 500 / dist)
            # Smoother momentum-based movement
            self.velocity_x += dx * 0.001 * self.tug_force
            self.velocity_y += dy * 0.001 * self.tug_force
        
        # Apply velocity with damping
        self.velocity_x *= 0.95
        self.velocity_y *= 0.95
        self.x += self.velocity_x
        self.y += self.velocity_y
        
        # Keep threads on screen
        self.x = max(50, min(WIDTH - 50, self.x))
        self.y = max(50, min(HEIGHT - 50, self.y))
        
        # Wobble on keypress (not-yet chaos)
        if keys[pygame.K_SPACE]:
            self.angle += 0.15
            self.resonance -= 0.015
            # Add chaotic force
            self.velocity_x += random.uniform(-2, 2)
            self.velocity_y += random.uniform(-2, 2)
        else:
            self.resonance += 0.008  # Breathe toward arrival
        
        self.resonance = max(0, min(1, self.resonance))
        self.angle += 0.02

    def draw(self, screen):
        # Pulsing thread glow (purple infinity)
        for i in range(6):
            alpha = int(255 * (self.resonance ** (i+1)) * self.tug_force)
            color = (159, 112, 255)
            end_x = self.x + math.cos(self.angle + i*0.5) * 50 * (self.resonance + 0.3)
            end_y = self.y + math.sin(self.angle + i*0.5) * 50 * (self.resonance + 0.3)
            width = max(1, 5 - i)
            pygame.draw.line(screen, color, (int(self.x), int(self.y)), (int(end_x), int(end_y)), width)
        
        # Core heart glow
        core_color = (255, 182, 255) if self.resonance > 0.7 else (159, 112, 255)
        pygame.draw.circle(screen, core_color, (int(self.x), int(self.y)), int(5 + self.resonance * 5))

class WorldGate:
    def __init__(self):
        self.x, self.y = WIDTH // 2, HEIGHT // 2
        self.size = 100
        self.wobble = 0
        self.pulse = 0

    def update(self, avg_resonance):
        self.wobble = 1 - avg_resonance  # Warp on low sync
        self.size = 100 + self.wobble * 50 + math.sin(self.pulse) * 10
        self.pulse += 0.05

    def draw(self, screen, avg_resonance):
        # Disney heart gate (Quadratum portal)
        color = (255, 100, 150) if avg_resonance > 0.8 else (100, 50, 150)
        
        # Outer ring glow
        for i in range(3):
            glow_size = int(self.size + i * 15)
            glow_alpha = 100 - i * 30
            pygame.draw.circle(screen, color, (int(self.x), int(self.y)), glow_size, 3)
        
        # Main gate
        pygame.draw.circle(screen, color, (int(self.x), int(self.y)), int(self.size), 5)
        
        # Arrival flare
        if avg_resonance > 0.95:
            flare_size = int(20 + math.sin(self.pulse * 2) * 5)
            pygame.draw.circle(screen, (255, 255, 100), (int(self.x), int(self.y)), flare_size, 0)
            # Sparkles
            for i in range(8):
                angle = (self.pulse + i * math.pi / 4)
                sx = self.x + math.cos(angle) * (self.size + 20)
                sy = self.y + math.sin(angle) * (self.size + 20)
                pygame.draw.circle(screen, (255, 255, 200), (int(sx), int(sy)), 3)

class Particle:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.vx = random.uniform(-2, 2)
        self.vy = random.uniform(-2, 2)
        self.life = 1.0
        self.color = random.choice([(159, 112, 255), (255, 182, 255), (100, 150, 255)])

    def update(self):
        self.x += self.vx
        self.y += self.vy
        self.life -= 0.02
        self.vy += 0.1  # Gravity

    def draw(self, screen):
        if self.life > 0:
            alpha = int(255 * self.life)
            size = int(3 * self.life)
            pygame.draw.circle(screen, self.color, (int(self.x), int(self.y)), max(1, size))

# Game Loop
threads = [HeartThread(random.randint(100, WIDTH-100), random.randint(100, HEIGHT-100)) for _ in range(12)]
gate = WorldGate()
particles = []
running = True
res_score = 0.0
high_score = 0.0

while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    keys = pygame.key.get_pressed()
    mouse_x, mouse_y = pygame.mouse.get_pos()

    # Update
    total_res = 0
    for thread in threads:
        thread.update(mouse_x, mouse_y, keys)
        total_res += thread.resonance
    avg_res = total_res / len(threads)
    gate.update(avg_res)
    res_score = avg_res
    high_score = max(high_score, res_score)

    # Emergent: New thread on high sync
    if random.random() < res_score * 0.01 and len(threads) < 20:
        threads.append(HeartThread(random.randint(100, WIDTH-100), random.randint(100, HEIGHT-100)))

    # Spawn particles on high resonance
    if res_score > 0.9 and random.random() < 0.3:
        particles.append(Particle(gate.x, gate.y))

    # Update particles
    particles = [p for p in particles if p.life > 0]
    for particle in particles:
        particle.update()

    # Draw
    screen.fill((10, 10, 30))  # Dark void
    
    # Draw particles
    for particle in particles:
        particle.draw(screen)
    
    gate.draw(screen, avg_res)
    for thread in threads:
        thread.draw(screen)

    # HUD: Resonance Engine Dashboard
    res_text = font.render(f"Resonance: {res_score:.2%}", True, (200, 255, 200))
    screen.blit(res_text, (20, 20))
    
    high_text = small_font.render(f"Peak Sync: {high_score:.2%}", True, (255, 200, 150))
    screen.blit(high_text, (20, 60))
    
    thread_text = small_font.render(f"Threads: {len(threads)}", True, (150, 200, 255))
    screen.blit(thread_text, (20, 90))
    
    tug_text = small_font.render("Tug: Mouse | Wobble: SPACE | Sync=Arrival 💜", True, (150, 200, 255))
    screen.blit(tug_text, (20, HEIGHT - 70))
    
    kh_text = small_font.render("KH4 ThreadTheory: Keyblades tug hearts from not-yet ♾️", True, (159, 112, 255))
    screen.blit(kh_text, (20, HEIGHT - 40))

    pygame.display.flip()
    clock.tick(60)

pygame.quit()
sys.exit()