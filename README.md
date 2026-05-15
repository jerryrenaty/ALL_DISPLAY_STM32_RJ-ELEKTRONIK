# Guide Complet du Driver d'Affichage RJ-ELEKTRONIK

## Table des Matières
1. [Introduction](#introduction)
2. [Installation et Configuration](#installation-et-configuration)
3. [Initialisation](#initialisation)
4. [Fonctions de Base](#fonctions-de-base)
5. [Dessin Géométrique](#dessin-géométrique)
6. [Affichage de Texte](#affichage-de-texte)
7. [Gestion des Images](#gestion-des-images)
8. [Mode DMA Haute Performance](#mode-dma-haute-performance)
9. [Interface Tactile (ILI9488)](#interface-tactile-ili9488)
10. [Exemples Complets](#exemples-complets)
11. [Dépannage](#dépannage)
12. [Optimisations](#optimisations)

## Introduction

Ce driver supporte 4 types d'écrans :
- **ST7789** : 240x320 ou 240x280 pixels
- **GC9A01** : 240x240 pixels (écran rond)
- **ILI9488** : 320x480 pixels (avec tactile)
- **ST7735** : 160x80 pixels

### Caractéristiques Principales
- ✅ Communication SPI haute vitesse
- ✅ Support DMA pour transferts non-bloquants
- ✅ Support DMA2D pour accélération matérielle (STM32H7)
- ✅ Gestion des rotations (0°, 90°, 180°, 270°)
- ✅ Interface tactile pour ILI9488
- ✅ Palette de 16 couleurs prédéfinies

## Installation et Configuration

### 1. Configuration dans CubeMX

```c
// Configuration SPI recommandée
SPI_HandleTypeDef hspi1;
hspi1.Instance = SPI1;
hspi1.Init.Mode = SPI_MODE_MASTER;
hspi1.Init.Direction = SPI_DIRECTION_2LINES_TXONLY;
hspi1.Init.DataSize = SPI_DATASIZE_8BIT;
hspi1.Init.CLKPolarity = SPI_POLARITY_HIGH;
hspi1.Init.CLKPhase = SPI_PHASE_2EDGE;
hspi1.Init.FirstBit = SPI_FIRSTBIT_MSB;
hspi1.Init.TIMode = SPI_TIMODE_DISABLE;
hspi1.Init.CRCCalculation = SPI_CRCCALCULATION_DISABLE;
hspi1.Init.NSS = SPI_NSS_SOFT;
hspi1.Init.BaudRatePrescaler = SPI_BAUDRATEPRESCALER_2;  // 36 MHz max
2. Configuration des Broches GPIO
Signal	GPIO	Description
CS	GPIOD, Pin 6	Chip Select (actif bas)
DC	GPIOD, Pin 4	Data/Command
RST	GPIOD, Pin 2	Reset
T_CS	GPIOB, Pin 12	Touch CS (ILI9488)
T_IRQ	GPIOA, Pin 8	Touch Interrupt (ILI9488)
3. Fichier de Configuration
Créez RJ-ELEKTRONIK_display_conf.h :

c
// Sélectionnez UN SEUL écran
#define Display_ILI9488      0  // Écran 320x480 tactile
#define Display_7789         1  // Écran 240x320/280
#define Display_GC9A01       0  // Écran rond 240x240
#define Display_ST7735       0  // Écran 160x80

// Configuration ST7789 (si sélectionné)
#if Display_7789
#define st7789_240x320    0    // 0=240x280, 1=240x320
#define GAMMA_OPTIMISÉE   1    // Améliore les couleurs
#endif

// Mode de transfert
#define USE_DMA_INTEGRATION    1  // 1=DMA, 0=Bloquant
#define USE_DMA2D              0  // Accélération DMA2D (STM32H7)

// Taille du buffer DMA (calculée automatiquement)
// Pour ST7789 240x280 : 2240 octets
// Pour ILI9488 : ~14KB
Initialisation
Initialisation Standard
c
#include "DISPLAYS_RJ-ELEKTRONIK.h"

// Handle SPI global (défini dans main.c)
extern SPI_HandleTypeDef hspi1;

int main(void) {
    // 1. Initialisation HAL
    HAL_Init();
    
    // 2. Configuration des horloges et GPIOs
    SystemClock_Config();
    MX_GPIO_Init();
    MX_SPI1_Init();
    
    // 3. Initialisation de l'écran
    Display_Init(&hspi1);
    
    // 4. Configuration de la rotation
    SetRotation(ROTATION_0);  // Portrait
    // SetRotation(ROTATION_90);  // Paysage
    
    // 5. Test rapide
    FillScreen(RED);
    HAL_Delay(1000);
    FillScreen(GREEN);
    HAL_Delay(1000);
    FillScreen(BLUE);
    
    while(1) {
        // Votre code
    }
}
Initialisation avec Tactile (ILI9488)
c
#if Display_ILI9488
// Handle SPI pour le tactile (généralement SPI2)
extern SPI_HandleTypeDef hspi2;

void init_display_with_touch(void) {
    // Initialiser l'affichage
    Display_Init(&hspi1);
    
    // Initialiser le tactile
    XPT2046_Init(&hspi2);
    
    // Lancer la calibration (une seule fois)
    XPT2046_Calibrate();
    
    // Afficher les résultats
    Display_Calibration_Results();
}
#endif
Fonctions de Base
Contrôle de l'Écran
c
// Rotation
SetRotation(ROTATION_0);    // Portrait normal
SetRotation(ROTATION_90);   // Paysage
SetRotation(ROTATION_180);  // Portrait inversé
SetRotation(ROTATION_270);  // Paysage inversé

// Dimensions dynamiques (après SetRotation)
uint16_t largeur = Display_Width;   // Ex: 240 en portrait
uint16_t hauteur = Display_Height;  // Ex: 320 en portrait

// Effacement
FillScreen(BLACK);   // Noir complet
FillScreen(WHITE);   // Blanc complet
Palette de Couleurs
c
// Pour ST7789/GC9A01/ST7735 (RGB565)
uint16_t colors_16bit[] = {
    BLACK,    // 0x0000
    WHITE,    // 0xFFFF
    RED,      // 0xF800
    GREEN,    // 0x07E0
    BLUE,     // 0x001F
    YELLOW,   // 0xFFE0
    CYAN,     // 0x07FF
    MAGENTA,  // 0xF81F
    ORANGE,   // 0xFD20
    PURPLE,   // 0x8010
    PINK,     // 0xFC1F
};

// Pour ILI9488 (RGB888)
uint32_t colors_24bit[] = {
    BLACK,    // 0x000000
    WHITE,    // 0xFFFFFF
    RED,      // 0xFF0000
    GREEN,    // 0x00FF00
    BLUE,     // 0x0000FF
    YELLOW,   // 0xFFFF00
    CYAN,     // 0x00FFFF
    MAGENTA,  // 0xFF00FF
};

// Utilisation
FillScreen(colors_16bit[2]);  // Remplir en rouge
Dessin Géométrique
Rectangles
c
// Rectangle plein
void FillRect(uint16_t x, uint16_t y, uint16_t w, uint16_t h, uint16_t color);
void FillRect(uint16_t x, uint16_t y, uint16_t w, uint16_t h, uint32_t color); // ILI9488

// Exemple : Cadre rouge au centre
int center_x = (Display_Width - 100) / 2;
int center_y = (Display_Height - 50) / 2;
FillRect(center_x, center_y, 100, 50, RED);
Bordures et Formes (À implémenter dans font.h)
c
// Ces fonctions doivent être dans votre font.h
extern void DrawRect(uint16_t x, uint16_t y, uint16_t w, uint16_t h, uint16_t color);
extern void DrawCircle(uint16_t x0, uint16_t y0, uint16_t radius, uint16_t color);
extern void DrawLine(uint16_t x1, uint16_t y1, uint16_t x2, uint16_t y2, uint16_t color);

// Exemple : Bordure bleue
DrawRect(0, 0, Display_Width-1, Display_Height-1, BLUE);

// Exemple : Horloge simple
DrawCircle(Display_Width/2, Display_Height/2, 100, WHITE);
DrawLine(Display_Width/2, Display_Height/2, 
         Display_Width/2 + 50 * cos(angle), 
         Display_Height/2 + 50 * sin(angle), RED);
Affichage de Texte
Polices Disponibles
Le driver utilise les polices définies dans font.h. Exemple avec Font5x7 :

c
void DrawString(uint16_t x, uint16_t y, 
                const char* text, 
                uint16_t foreground, 
                uint16_t background, 
                uint8_t scale, 
                const FontDef_t* font);

// Paramètres :
// - x, y : Position de départ (coin supérieur gauche)
// - text : Chaîne à afficher
// - foreground : Couleur du texte
// - background : Couleur du fond
// - scale : Facteur d'agrandissement (1=taille normale, 2=double)
// - font : Pointeur vers la police
Exemples d'Affichage Texte
c
// 1. Texte simple
DrawString(10, 10, "Hello World!", WHITE, BLACK, 1, &Font5x7);

// 2. Titre en grand
DrawString(50, 100, "TITRE", YELLOW, BLUE, 2, &Font5x7);

// 3. Variables formatées
char buffer[32];
int temperature = 25;
sprintf(buffer, "Temp: %d°C", temperature);
DrawString(10, 200, buffer, GREEN, BLACK, 1, &Font5x7);

// 4. Menu avec surbrillance
DrawString(20, 50, "> Option 1", WHITE, BLACK, 1, &Font5x7);
DrawString(20, 70, "  Option 2", GRAY, BLACK, 1, &Font5x7);
DrawString(20, 90, "  Option 3", GRAY, BLACK, 1, &Font5x7);

// 5. Centrage automatique
uint16_t text_width = strlen("Message") * 5 * 1; // 5 pixels par char en Font5x7
uint16_t text_x = (Display_Width - text_width) / 2;
DrawString(text_x, Display_Height/2, "Message", WHITE, BLACK, 1, &Font5x7);
Gestion des Images
Préparation des Images
Convertir votre image en format RGB565 (ou RGB888 pour ILI9488)

Utiliser un outil comme Image2C, LCD Image Converter, ou GIMP

Exporter en tableau C :

c
// Exemple pour image 50x50 pixels en RGB565
const uint16_t my_image[2500] = {
    0xFFFF, 0xFFFF, 0xF800, /* ... */
    // 2500 pixels au total
};
Affichage d'Images
c
// Mode SPI Bloquant
#if USE_SPI_BLOQUANT
void DrawBitmap(uint16_t x, uint16_t y, 
                const uint16_t *bitmap, 
                uint16_t w, uint16_t h);

// Mode DMA
#if USE_DMA_INTEGRATION
void DrawBitmap(uint16_t x, uint16_t y, 
                const uint16_t *bitmap, 
                uint16_t w, uint16_t h);
#endif

// Exemple
extern const uint16_t logo[1600];  // 40x40 pixels
DrawBitmap(100, 100, logo, 40, 40);
Animation Image par Image
c
const uint16_t* frames[] = {frame1, frame2, frame3, frame4};
int current_frame = 0;
int num_frames = 4;

while(1) {
    DrawBitmap(0, 0, frames[current_frame], 50, 50);
    current_frame = (current_frame + 1) % num_frames;
    HAL_Delay(100);  // 10 fps
}
Mode DMA Haute Performance
Configuration DMA
c
// Dans le fichier de configuration
#define USE_DMA_INTEGRATION 1
#define D_CACHE_CORTEX_M7 1  // Pour STM32H7

// Tampon DMA (déclaré automatiquement)
extern uint8_t dma_buffer[DMA_BUFFER_SIZE];
Utilisation du DMA
c
// Les mêmes fonctions, mais non-bloquantes !
FillScreen(RED);  // Transfert DMA en arrière-plan

// Vérifier si le DMA est occupé
if (!DMA_IsBusy()) {
    // Le DMA a fini, on peut dessiner autre chose
    DrawString(10, 10, "DMA Pret", GREEN, BLACK, 1, &Font5x7);
}

// Attendre la fin (bloquant)
WaitDMA();  // À utiliser avant de changer le buffer DMA
Performance DMA vs Bloquant
c
// Mesure de performance
uint32_t start = HAL_GetTick();
FillRect(0, 0, Display_Width, Display_Height, RED);
uint32_t dma_time = HAL_GetTick() - start;
// Le DMA retourne IMMÉDIATEMENT (temps ~0ms)

start = HAL_GetTick();
WaitDMA();  // Attendre la fin réelle
uint32_t real_time = HAL_GetTick() - start;
// real_time = temps réel du transfert (~10-50ms selon taille)
Double Buffering pour Animations Fluides
c
#define NUM_BUFFERS 2
uint8_t buffers[NUM_BUFFERS][DMA_BUFFER_SIZE];
int current_buffer = 0;

void render_frame(int buffer_idx) {
    uint16_t* fb = (uint16_t*)buffers[buffer_idx];
    // Dessiner dans le framebuffer
    for(int i = 0; i < (Display_Width * Display_Height); i++) {
        fb[i] = calculate_pixel_color(i);
    }
    
    // Envoyer via DMA
    WaitDMA();  // Attendre que l'écran soit prêt
    SetWindow(0, 0, Display_Width-1, Display_Height-1);
    CS_Low();
    DC_Data();
    HAL_SPI_Transmit_DMA(hspi, buffers[buffer_idx], 
                         Display_Width * Display_Height * 2);
}

void animation_loop(void) {
    while(1) {
        render_frame(current_buffer);
        current_buffer = (current_buffer + 1) % NUM_BUFFERS;
        // CPU peut préparer la prochaine frame pendant le DMA
    }
}
Cas Spécial ILI9488 - Zoom Matériel
c
#if Display_ILI9488
// Fonction spéciale pour zoomer une image 320x240 sur 480x320
// Utile pour les caméras ou images plus petites
void DrawBitmapFullZoom(const uint16_t *bitmap);

// Exemple avec image QVGA (320x240)
extern const uint16_t camera_image[76800];  // 320*240
DrawBitmapFullZoom(camera_image);  // Affiche en plein écran avec zoom x1.5
#endif
Interface Tactile (ILI9488)
Initialisation du Tactile
c
#if Display_ILI9488
SPI_HandleTypeDef hspi2;  // SPI pour le tactile

void touch_init(void) {
    // Initialiser le bus SPI pour le tactile
    MX_SPI2_Init();
    
    // Initialiser le driver
    XPT2046_Init(&hspi2);
    
    // Calibrer (à faire une fois, sauvegarder les coefficients)
    XPT2046_Calibrate();
    
    // Afficher les résultats
    Display_Calibration_Results();
}
#endif
Lecture des Coordonnées Tactiles
c
#if Display_ILI9488
TouchPoint point;
char buffer[32];

while(1) {
    if (XPT2046_GetPointSafe(&point)) {
        // Le doigt est posé sur l'écran
        sprintf(buffer, "X:%4d Y:%4d", point.x, point.y);
        DrawString(10, 10, buffer, WHITE, BLACK, 1, &Font5x7);
        
        // Dessiner un point à l'emplacement du doigt
        FillRect(point.x-2, point.y-2, 5, 5, RED);
        
        // Détection de zone
        if (point.x > 100 && point.x < 200 && point.y > 100 && point.y < 150) {
            DrawString(10, 30, "Bouton presse!", GREEN, BLACK, 1, &Font5x7);
        }
    } else {
        // Aucun contact
        DrawString(10, 10, "En attente...   ", WHITE, BLACK, 1, &Font5x7);
    }
    
    HAL_Delay(20);  // Éviter de surcharger le CPU
}
#endif
Création de Boutons Tactiles
c
#if Display_ILI9488
typedef struct {
    uint16_t x, y, w, h;
    const char* label;
    uint16_t color_normal;
    uint16_t color_pressed;
    void (*callback)(void);
} TouchButton;

TouchButton buttons[] = {
    {10, 10, 100, 50, "START", GREEN, RED, start_function},
    {120, 10, 100, 50, "STOP", RED, YELLOW, stop_function},
    {230, 10, 100, 50, "RESET", BLUE, WHITE, reset_function},
};

void draw_button(TouchButton* btn, int pressed) {
    uint16_t color = pressed ? btn->color_pressed : btn->color_normal;
    FillRect(btn->x, btn->y, btn->w, btn->h, color);
    DrawRect(btn->x, btn->y, btn->w, btn->h, WHITE);
    
    uint16_t text_x = btn->x + (btn->w - strlen(btn->label) * 6) / 2;
    uint16_t text_y = btn->y + (btn->h - 8) / 2;
    DrawString(text_x, text_y, btn->label, BLACK, color, 1, &Font5x7);
}

void handle_touch_buttons(TouchPoint* point) {
    for(int i = 0; i < 3; i++) {
        if(point->x >= buttons[i].x && point->x <= buttons[i].x + buttons[i].w &&
           point->y >= buttons[i].y && point->y <= buttons[i].y + buttons[i].h) {
            draw_button(&buttons[i], 1);
            if(buttons[i].callback) buttons[i].callback();
            HAL_Delay(200);  // Anti-rebond
            draw_button(&buttons[i], 0);
        }
    }
}

// Boucle principale
while(1) {
    TouchPoint point;
    if (XPT2046_GetPointSafe(&point)) {
        handle_touch_buttons(&point);
    }
}
#endif
Exemples Complets
Exemple 1 : Interface Météo Simple
c
#include "DISPLAYS_RJ-ELEKTRONIK.h"
#include <stdlib.h>

// Variables météo simulées
int temperature = 22;
int humidity = 65;
int pressure = 1013;

void draw_weather_interface(void) {
    char buffer[32];
    
    // Fond
    FillScreen(BLACK);
    
    // Titre
    DrawString(60, 10, "STATION METEO", CYAN, BLACK, 2, &Font5x7);
    
    // Ligne de séparation
    FillRect(0, 35, Display_Width, 2, WHITE);
    
    // Température
    sprintf(buffer, "Temp: %d C", temperature);
    DrawString(20, 60, buffer, RED, BLACK, 1, &Font5x7);
    
    // Jauge température
    FillRect(20, 80, temperature * 3, 20, RED);
    DrawRect(20, 80, 150, 20, WHITE);
    
    // Humidité
    sprintf(buffer, "Hum: %d%%", humidity);
    DrawString(20, 120, buffer, BLUE, BLACK, 1, &Font5x7);
    
    // Jauge humidité
    FillRect(20, 140, humidity * 1.5, 20, BLUE);
    DrawRect(20, 140, 100, 20, WHITE);
    
    // Pression
    sprintf(buffer, "Press: %d hPa", pressure);
    DrawString(20, 180, buffer, GREEN, BLACK, 1, &Font5x7);
    
    // Heure
    sprintf(buffer, "%02d:%02d:%02d", 
            HAL_GetTick() / 3600000 % 24,
            HAL_GetTick() / 60000 % 60,
            HAL_GetTick() / 1000 % 60);
    DrawString(Display_Width - 80, Display_Height - 20, buffer, WHITE, BLACK, 1, &Font5x7);
}

int main(void) {
    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init();
    MX_SPI1_Init();
    
    Display_Init(&hspi1);
    SetRotation(ROTATION_0);
    
    while(1) {
        draw_weather_interface();
        
        // Simuler changement des valeurs
        temperature = 20 + rand() % 15;
        humidity = 40 + rand() % 50;
        pressure = 1000 + rand() % 30;
        
        HAL_Delay(2000);  // Rafraîchir toutes les 2 secondes
    }
}
Exemple 2 : Animation avec DMA
c
#if USE_DMA_INTEGRATION
#include "math.h"

// Effet vague avec sinus
void draw_wave_animation(void) {
    static float phase = 0;
    uint16_t wave_buffer[Display_Width];
    
    // Préparer la ligne courante
    for(uint16_t x = 0; x < Display_Width; x++) {
        int16_t y = 50 + 30 * sin(x * 0.05 + phase);
        wave_buffer[x] = (uint16_t)y;
    }
    
    // Dessiner vague
    SetWindow(0, 0, Display_Width-1, Display_Height-1);
    CS_Low();
    DC_Data();
    
    for(uint16_t y = 0; y < Display_Height; y++) {
        // Convertir la ligne en pixels
        for(uint16_t x = 0; x < Display_Width; x++) {
            uint16_t color = (abs(y - wave_buffer[x]) < 5) ? RED : BLACK;
            dma_buffer[x*2] = color >> 8;
            dma_buffer[x*2+1] = color & 0xFF;
        }
        
        WaitDMA();
        HAL_SPI_Transmit_DMA(hspi, dma_buffer, Display_Width * 2);
    }
    CS_High();
    
    phase += 0.1;
}

int main(void) {
    Display_Init(&hspi1);
    SetRotation(ROTATION_0);
    
    while(1) {
        draw_wave_animation();
        HAL_Delay(50);
    }
}
#endif
Exemple 3 : Éditeur de Formes avec Tactile
c
#if Display_ILI9488
typedef enum {MODE_RECT, MODE_CIRCLE} DrawMode;
DrawMode current_mode = MODE_RECT;
uint16_t start_x, start_y;
int drawing = 0;

void handle_touch_drawing(TouchPoint* point) {
    if(!drawing) {
        // Premier contact : début de la forme
        start_x = point->x;
        start_y = point->y;
        drawing = 1;
    } else {
        // Glisser : afficher l'aperçu
        FillRect(0, 200, Display_Width, 80, BLACK);  // Effacer zone info
        
        char buffer[64];
        sprintf(buffer, "Largeur: %d, Hauteur: %d", 
                abs(point->x - start_x), abs(point->y - start_y));
        DrawString(10, 210, buffer, WHITE, BLACK, 1, &Font5x7);
        
        // Aperçu transparent
        if(current_mode == MODE_RECT) {
            DrawRect(start_x, start_y, point->x - start_x, point->y - start_y, YELLOW);
        } else {
            int radius = abs(point->x - start_x) / 2;
            DrawCircle(start_x, start_y, radius, YELLOW);
        }
    }
}

void finalize_shape(TouchPoint* point) {
    if(current_mode == MODE_RECT) {
        FillRect(start_x, start_y, point->x - start_x, point->y - start_y, BLUE);
    } else {
        int radius = abs(point->x - start_x) / 2;
        DrawCircle(start_x, start_y, radius, RED);
    }
    drawing = 0;
}

int main(void) {
    Display_Init(&hspi1);
    XPT2046_Init(&hspi2);
    XPT2046_Calibrate();
    SetRotation(ROTATION_0);
    
    // Interface utilisateur
    FillScreen(BLACK);
    DrawString(10, 10, "Appuyez et glissez pour dessiner", WHITE, BLACK, 1, &Font5x7);
    DrawString(10, 30, "Mode: RECTANGLE", YELLOW, BLACK, 1, &Font5x7);
    DrawString(10, 50, "Appuyez sur le bouton tactile", WHITE, BLACK, 1, &Font5x7);
    
    while(1) {
        TouchPoint point;
        if(XPT2046_GetPointSafe(&point)) {
            if(!drawing) {
                handle_touch_drawing(&point);
            } else {
                // Vérifier si le doigt est relâché
                if(!XPT2046_IsPressed()) {
                    finalize_shape(&point);
                } else {
                    handle_touch_drawing(&point);
                }
            }
        }
        HAL_Delay(20);
    }
}
#endif
Dépannage
Problèmes Courants et Solutions
Problème	Cause Possible	Solution
Écran blanc	Mauvaise initialisation	Vérifier les connexions GPIO, augmenter délais reset
Couleurs inversées	Mauvais format de pixel	Vérifier WriteData(0x55) pour RGB565
Image décalée	Mauvais offsets	Ajuster ST7789_X_Offset et ST7789_Y_Offset
DMA ne fonctionne pas	Cache non nettoyé	Activer D_CACHE_CORTEX_M7
Tactile inexact	Mauvaise calibration	Recalibrer avec XPT2046_Calibrate()
SPI timeout	Vitesse trop élevée	Réduire prescaler SPI
Debug avec Sortie Série
c
#include <stdio.h>

void display_debug_info(void) {
    char buffer[128];
    
    sprintf(buffer, "Display: %dx%d", Display_Width, Display_Height);
    DrawString(10, 10, buffer, WHITE, BLACK, 1, &Font5x7);
    
    sprintf(buffer, "Rotation: %d", currentRotation);
    DrawString(10, 30, buffer, WHITE, BLACK, 1, &Font5x7);
    
    #if USE_DMA_INTEGRATION
    sprintf(buffer, "DMA: %s", DMA_IsBusy() ? "BUSY" : "IDLE");
    DrawString(10, 50, buffer, WHITE, BLACK, 1, &Font5x7);
    #endif
    
    #if Display_ILI9488
    sprintf(buffer, "Touch: %s", XPT2046_IsPressed() ? "PRESSED" : "RELEASED");
    DrawString(10, 70, buffer, WHITE, BLACK, 1, &Font5x7);
    #endif
}
Optimisations
1. Réduction de la Consommation Mémoire
c
// Utiliser des polices plus petites
#define USE_SMALL_FONT 1

// Limiter la taille du buffer DMA
#if defined(STM32F103xB)  // Petit MCU
    #define DMA_BUFFER_SIZE 1024
#else
    #define DMA_BUFFER_SIZE 4096
#endif
2. Amélioration des Performances
c
// Regrouper les commandes SPI
void optimized_draw(void) {
    CS_Low();
    DC_Data();
    
    // Envoyer tout en une seule fois
    HAL_SPI_Transmit(hspi, huge_buffer, huge_size, HAL_MAX_DELAY);
    
    CS_High();
}

// Utiliser le DMA2D pour STM32H7
#if USE_DMA2D
    // Les remplissages sont 10x plus rapides
    FillScreen(RED);  // Utilise DMA2D automatiquement
#endif
3. Gestion Énergétique
c
void display_sleep(void) {
    WriteCommand(0x10);  // Mode sommeil
    HAL_GPIO_WritePin(RST_GPIO_Port, RST_Pin, GPIO_PIN_RESET);
}

void display_wakeup(void) {
    HAL_GPIO_WritePin(RST_GPIO_Port, RST_Pin, GPIO_PIN_SET);
    HAL_Delay(100);
    Display_Init(hspi);  // Réinitialisation complète
}
Conclusion
Ce driver offre une interface complète et performante pour les écrans TFT SPI. Les points clés à retenir :

Sélectionnez un seul écran dans la configuration

Activez le DMA pour les animations fluides

Calibrez le tactile pour ILI9488

Utilisez WaitDMA() avant de modifier les buffers DMA

Testez avec les exemples fournis avant d'intégrer votre code

Pour toute question ou bug, consultez la section Dépannage ou contactez RJ-ELEKTRONIK.

Documentation Version 1.0 - © 2026 RJ-ELEKTRONIK

text

Ce fichier Markdown complet couvre :
- ✅ Installation et configuration détaillée
- ✅ Toutes les fonctions avec exemples
- ✅ Mode SPI bloquant et DMA
- ✅ Interface tactile complète
- ✅ 3 exemples complets utilisables directement
- ✅ Dépannage et optimisations
- ✅ Bonnes pratiques de programmation

Vous pouvez sauvegarder ce contenu dans un fichier `README.md` ou `USER_GUIDE.md` à la racine de votre projet.
