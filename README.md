import pygame
import math
import random
import sys

pygame.init()
WIDTH, HEIGHT = 950, 950
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Gargantua: Multilingual Love Meteor Vortex ✨🖤")
clock = pygame.time.Clock()

# --- BẢNG MÀU CHIỀU KHÔNG GIAN ---
COLOR_THEMES = [
    ("Interstellar Gold 🔥", (255, 245, 190), (255, 140, 30), (255, 70, 15)),
    ("Cyberpunk Nebula 💜", (255, 200, 255), (195, 55, 255), (0, 245, 255)),
    ("Quantum Horizon 💎", (215, 255, 255), (0, 190, 255), (70, 70, 255)),
    ("Solar Flare Blood ❤️", (255, 225, 215), (255, 50, 80), (160, 0, 40))
]
current_theme_idx = 0

# --- CÁC NGÔN NGỮ TÌNH YÊU ---
LOVE_LANGUAGES = [
    "I LOVE YOU",
    "ANH YÊU EM",
    "EM YÊU ANH",
    "TE AMO",
    "JE T'AIME",
    "SARANGHAE",
    "AISHITERU",
    "WO AI NI",
    "TI AMO",
    "ICH LIEBE DICH"
]

font_ui = pygame.font.SysFont("trebuchetms", 15)
font_title = pygame.font.SysFont("georgia", 24, bold=True)
font_love = pygame.font.SysFont("arialblack", 13)

# -------------------------------------------------------------
# 1. TOÁN HỌC TRÁI TIM CHÂN TRỜI SỰ KIỆN
# -------------------------------------------------------------
def get_heart_target(t, scale=5.0):
    x = 16 * (math.sin(t) ** 3)
    y = -(13 * math.cos(t) - 5 * math.cos(2 * t) - 2 * math.cos(3 * t) - math.cos(4 * t))
    return x * scale, y * scale

# -------------------------------------------------------------
# 2. HỆ THỐNG SAO BĂNG (SHOOTING STARS)
# -------------------------------------------------------------
class ShootingStar:
    def __init__(self):
        self.reset()

    def reset(self):
        # Xuất phát từ rìa trên / phải
        if random.random() < 0.6:
            self.x = random.randint(150, WIDTH + 100)
            self.y = random.randint(-50, 80)
        else:
            self.x = random.randint(WIDTH - 100, WIDTH + 120)
            self.y = random.randint(0, HEIGHT // 2)

        angle = random.uniform(math.radians(135), math.radians(158))
        speed = random.uniform(14, 21)
        self.vx = math.cos(angle) * speed
        self.vy = math.sin(angle) * speed
        self.length = random.uniform(70, 130)
        self.active = False
        self.wait_time = random.uniform(0.1, 1.😎

    def update(self, dt):
        if not self.active:
            self.wait_time -= dt
            if self.wait_time <= 0:
                self.active = True
        else:
            self.x += self.vx
            self.y += self.vy
            if self.x < -150 or self.y > HEIGHT + 100:
                self.reset()

    def draw(self, surface, tail_color):
        if not self.active:
            return
        speed_norm = math.hypot(self.vx, self.vy)
        tail_x = self.x - (self.vx / speed_norm) * self.length
        tail_y = self.y - (self.vy / speed_norm) * self.length

        pygame.draw.line(surface, tail_color, (int(self.x), int(self.y)), (int(tail_x), int(tail_y)), 2)
        pygame.draw.circle(surface, (255, 255, 255), (int(self.x), int(self.y)), 2)

# -------------------------------------------------------------
# 3. CHỮ TÌNH YÊU ĐA NGÔN NGỮ BAY XÉO XUNG QUANH
# -------------------------------------------------------------
class FloatingLoveWord:
    def __init__(self):
        self.reset(initial=True)

    def reset(self, initial=False):
        self.text = random.choice(LOVE_LANGUAGES)
        
        # Bay chéo theo góc ngẫu nhiên (chủ yếu từ góc dưới-trái lên trên-phải hoặc trên-phải xuống)
        angle = random.uniform(math.radians(-55), math.radians(-25)) if random.random() < 0.5 \
            else random.uniform(math.radians(125), math.radians(155))
            
        speed = random.uniform(1.2, 2.6)
        self.vx = math.cos(angle) * speed
        self.vy = math.sin(angle) * speed

        if initial:
            self.x = random.randint(50, WIDTH - 50)
            self.y = random.randint(80, HEIGHT - 80)
        else:
            # Sinh từ các góc rìa màn hình
            if self.vx > 0:
                self.x = random.randint(-150, -30)
                self.y = random.randint(100, HEIGHT - 50)
            else:
                self.x = random.randint(WIDTH + 30, WIDTH + 150)
                self.y = random.randint(50, HEIGHT - 150)

        self.alpha = random.randint(140, 225)
        self.scale_pulse = random.uniform(0.9, 1.2)
        self.rot_deg = math.degrees(angle) * 0.4  # Nghiêng nhẹ theo hướng bay

    def update(self):
        self.x += self.vx
        self.y += self.vy

        # Bay khỏi vùng hiển thị thì tái tạo lại
        if self.x < -180 or self.x > WIDTH + 180 or self.y < -80 or self.y > HEIGHT + 80:
            self.reset(initial=False)

    def draw(self, surface, color):
        txt_surf = font_love.render(self.text, True, color)
        txt_surf.set_alpha(int(self.alpha))
        
        # Xoay chữ theo góc nghiêng bay
        rotated = pygame.transform.rotate(txt_surf, self.rot_deg)
        rect = rotated.get_rect(center=(int(self.x), int(self.y)))
        surface.blit(rotated, rect)

# -------------------------------------------------------------
# 4. SÓNG XUNG KÍCH TRÁI TIM TỪ TÂM (RIPPLE SHOCKWAVE)
# -------------------------------------------------------------
class HeartShockwave:
    def __init__(self, cx, cy, color, start_scale=4.5):
        self.cx = cx
        self.cy = cy
        self.color = color
        self.scale = start_scale
        self.max_scale = 32.0
        self.alpha = 240.0
        self.speed = 0.45
        self.dead = False

    def update(self, dt):
        self.scale += self.speed * 60.0 * dt
        progress = (self.scale - 4.5) / (self.max_scale - 4.5)
        self.alpha = max(0.0, 240.0 * (1.0 - progress ** 0.8))
        if self.scale >= self.max_scale or self.alpha <= 0:
            self.dead = True

    def draw(self, surface):
        if self.dead or self.alpha <= 2:
            return

        pts = []
        for deg in range(0, 360, 6):
            rad = math.radians(deg)
            hx, hy = get_heart_target(rad, scale=self.scale)
            pts.append((int(self.cx + hx), int(self.cy + hy * 0.58)))

        if len(pts) > 2:
            wave_surf = pygame.Surface((WIDTH, HEIGHT), pygame.SRCALPHA)
            a_int = int(self.alpha)
            
            glow_col = (*self.color, max(0, a_int // 3))
            pygame.draw.lines(wave_surf, glow_col, True, pts, 5)
            
            core_col = (255, 255, 255, a_int)
            pygame.draw.lines(wave_surf, core_col, True, pts, 2)
            
            surface.blit(wave_surf, (0, 0))

# -------------------------------------------------------------
# 5. HẠT PHOTON TRÊN ĐĨA BỒI TỤ (PHOTON DISK PARTICLES)
# -------------------------------------------------------------
class PhotonParticle:
    def __init__(self, cx, cy):
        self.cx = cx
        self.cy = cy
        self.reset(initial=True)

    def reset(self, initial=False):
        self.t = random.uniform(0, math.pi * 2)
        self.radius_layer = random.uniform(0.8, 3.2) if not initial else random.uniform(0.4, 3.4)
        self.orbit_speed = random.uniform(0.02, 0.055) / (self.radius_layer ** 0.5)
        
        self.tilt = 0.58
        self.z_drift = random.uniform(-16, 16)
        
        self.prev_x = None
        self.prev_y = None
        self.vx = 0.0
        self.vy = 0.0
        self.size = random.uniform(1.2, 2.5)

    def update(self, dt, current_time, mouse_pos, is_collapsing, supernova_active):
        self.t = (self.t + self.orbit_speed) % (math.pi * 2)
        hx, hy = get_heart_target(self.t, scale=5.0 * self.radius_layer)
        
        tx = self.cx + hx
        ty = self.cy + (hy * math.cos(self.tilt) - self.z_drift * math.sin(self.tilt))
        
        mx, my = mouse_pos
        dx = mx - tx
        dy = my - ty
        dist_sq = dx * dx + dy * dy + 350.0
        dist = math.sqrt(dist_sq)

        if is_collapsing:
            pull = 7500.0 / dist_sq
            self.vx += (dx / dist) * pull * 1.8
            self.vy += (dy / dist) * pull * 1.8
        elif supernova_active:
            push = 16000.0 / dist_sq
            self.vx -= (dx / dist) * push
            self.vy -= (dy / dist) * push
        else:
            if dist < 200:
                swirl_force = (200 - dist) * 0.05
                self.vx += (-dy / dist) * swirl_force
                self.vy += (dx / dist) * swirl_force

        self.vx *= 0.90
        self.vy *= 0.90

        final_x = tx + self.vx
        final_y = ty + self.vy
        return final_x, final_y

# -------------------------------------------------------------
# 6. KHỞI TẠO VŨ TRỤ VÀ VÒNG LẶP CHÍNH
# -------------------------------------------------------------
CENTER_X = WIDTH // 2
CENTER_Y = HEIGHT // 2

NUM_PHOTONS = 2400
photons = [PhotonParticle(CENTER_X, CENTER_Y) for _ in range(NUM_PHOTONS)]

# Sao băng bay chéo
shooting_stars = [ShootingStar() for _ in range(5)]

# Chữ tình yêu đa ngôn ngữ bay xéo xung quanh
love_words = [FloatingLoveWord() for _ in range(24)]

shockwaves = []
shockwave_timer = 0.0

bg_stars = [
    (random.randint(0, WIDTH), random.randint(0, HEIGHT), random.uniform(0.6, 2.0), random.uniform(0.01, 0.04), random.uniform(0, math.pi * 2))
    for _ in range(160)
]

blur_surface = pygame.Surface((WIDTH, HEIGHT))
blur_surface.set_alpha(52)
blur_surface.fill((4, 3, 10))

running = True
is_collapsing = False
supernova_timer = 0

while running:
    dt = clock.tick(60) / 1000.0
    current_time = pygame.time.get_ticks() / 1000.0

    mouse_pressed = pygame.mouse.get_pressed()[0]
    mouse_pos = pygame.mouse.get_pos()

    for event in pygame.event.get():
        if event.type == pygame.QUIT or (event.type == pygame.KEYDOWN and event.key == pygame.K_ESCAPE):
            running = False
        elif event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
            current_theme_idx = (current_theme_idx + 1) % len(COLOR_THEMES)
        elif event.type == pygame.MOUSEBUTTONUP and event.button == 1:
            supernova_timer = 18
            shockwaves.append(HeartShockwave(CENTER_X, CENTER_Y, (255, 255, 255), start_scale=4.5))

    is_collapsing = mouse_pressed
    if supernova_timer > 0:
        supernova_timer -= 1
        supernova_active = True
    else:
        supernova_active = False

    # 1. Phủ lớp mờ Motion Blur để giữ vệt đuôi ánh sáng
    screen.blit(blur_surface, (0, 0))

    theme_name, col_core, col_mid, col_outer = COLOR_THEMES[current_theme_idx]

    # 2. Vẽ sao nền nhấp nháy
    for sx, sy, s_rad, s_spd, s_phase in bg_stars:
        s_alpha = int((math.sin(current_time * s_spd * 20.0 + s_phase) + 1) * 70 + 40)
        pygame.draw.circle(screen, (190, 210, 240, s_alpha), (sx, sy), int(s_rad))

    # 3. Cập nhật và vẽ sao băng
    for ss in shooting_stars:
        ss.update(dt)
        ss.draw(screen, col_core)

    # 4. Kích hoạt và vẽ sóng xung kích từ tâm
    shockwave_timer += dt
    if shockwave_timer > 0.75:
        shockwaves.append(HeartShockwave(CENTER_X, CENTER_Y, col_mid))
        shockwave_timer = 0.0

    for sw in shockwaves[:]:
        sw.update(dt)
        sw.draw(screen)
        if sw.dead:
            shockwaves.remove(sw)

    # 5. Cập nhật và vẽ các chữ tình yêu đa ngôn ngữ bay xéo
    for word in love_words:
        word.update()
        word.draw(screen, col_core)

    # 6. Render đĩa bồi tụ hạt photon
    for p in photons:
        curr_x, curr_y = p.update(dt, current_time, mouse_pos, is_collapsing, supernova_active)

        if p.prev_x is not None:
            if p.radius_layer < 1.2:
                color = col_core
            elif p.radius_layer < 2.2:
                color = col_mid
            else:
                color = col_outer

            pygame.draw.line(screen, color, (int(p.prev_x), int(p.prev_y)), (int(curr_x), int(curr_y)), int(p.size))

        p.prev_x = curr_x
        p.prev_y = curr_y

    # 7. Vẽ chân trời sự kiện (Trái tim đen tại tâm)
    heartbeat_pulse = 1.0 + math.sin(current_time * 4.2) * 0.06
    event_horizon_pts = []
    for deg in range(0, 360, 6):
        rad_val = math.radians(deg)
        hx, hy = get_heart_target(rad_val, scale=4.2 * heartbeat_pulse)
        px = CENTER_X + hx
        py = CENTER_Y + hy * 0.58
        event_horizon_pts.append((px, py))

    pygame.draw.polygon(screen, (0, 0, 0), event_horizon_pts)
    pygame.draw.lines(screen, col_core, True, event_horizon_pts, 3)

    # 8. Hiệu ứng thấu kính khi giữ chuột
    if is_collapsing:
        pulse_r = int(22 + math.sin(current_time * 20.0) * 7)
        pygame.draw.circle(screen, col_core, mouse_pos, pulse_r, 2)
        pygame.draw.circle(screen, col_mid, mouse_pos, pulse_r + 12, 1)

    # 9. Giao diện HUD
    title_surf = font_title.render("MULTILINGUAL LOVE GRAVITATIONAL VORTEX", True, col_core)
    screen.blit(title_surf, (WIDTH // 2 - title_surf.get_width() // 2, 35))

    desc1 = font_ui.render("Di chuột: Uốn cong dòng hạt | Giữ chuột: Hút sụp đổ | Thả chuột: Nổ Supernova", True, (220, 220, 235))
    desc2 = font_ui.render(f"Phím [ SPACE ]: Chiều không gian — Hiện tại: {theme_name}", True, col_mid)
    screen.blit(desc1, (WIDTH // 2 - desc1.get_width() // 2, HEIGHT - 65))
    screen.blit(desc2, (WIDTH // 2 - desc2.get_width() // 2, HEIGHT - 40))

    pygame.display.flip()

pygame.quit()
sys.exit()
