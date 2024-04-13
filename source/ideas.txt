#include <tonc.h>

#define M3_WIDTH   SCREEN_WIDTH
typedef COLOR      M3LINE[M3_WIDTH];
#define m3_mem   ((M3LINE*)MEM_VRAM)

#define XMAX 239
#define YMAX 159
#define CLR_BLK 0x0000
#define CLR_RED 0x001F
#define CLR_YEL 0x03FF
#define CLR_GRN 0x03E0
#define CLR_CYN 0x7FE0
#define CLR_BLU 0x7C00
#define CLR_MAG 0x7C1F
#define CLR_WHT 0x7FFF
#define CLR_GRY 0x4210

// bugs
#define SPECIES 6
#define MAXBUGS 255

// foods
#define MAXFOODS 4
#define SPLOTCH 7
#define SPLOFF 3
#define ENERGY_INIT 370



/*****************************************************************************\
 Bacteria
\*****************************************************************************/

struct bacterium {
	u8  x;    // 1 .. XMAX-1
	u8  y;    // 1 .. YMAX-1
	u8  dir;  // directions: 1 .. 8
	u8  mem;  // sensor memory: 0 .. 255
	u16 sto;  // stored energy: 0 .. 200
	u16 col;  // COLOR
};
typedef struct bacterium bug;

bug new_bug(u8 x, u8 y, u8 species, u16 energy) {
	bug b;
	b.x = (x == 255) ? qran_range(0, XMAX) : x;
	b.y = (y == 255) ? qran_range(0, YMAX) : y;
	b.dir = qran_range(1, 8);
	b.mem = 0;
	b.sto = energy;
	switch (species) {
		case 0: b.col = CLR_RED; break;
		case 1: b.col = CLR_YEL; break;
		case 2: b.col = CLR_GRN; break;
		case 3: b.col = CLR_CYN; break;
		case 4: b.col = CLR_BLU; break;
		case 5: b.col = CLR_MAG; break;
		case 6: b.col = CLR_WHT; break;
	}
	return b;
}

void update_bugs(bug *bugs) {
	// thermal vibration
	for (int i = 0; i < MAXBUGS; i++) {
		bugs[i].x += qran_range(0, 3) - 1;
		bugs[i].y += qran_range(0, 3) - 1;
		if (bugs[i].x < 1) bugs[i].x = 1;
		if (bugs[i].x > XMAX-1) bugs[i].x = XMAX-1;
		if (bugs[i].y < 1) bugs[i].y = 1;
		if (bugs[i].y > YMAX-1) bugs[i].y = YMAX-1;
	}

	// chemotaxis

	// energy gain

	// energy loss

	// reproduction

	// unplot the bug

	// plot the bug
	for (int i = 0; i < MAXBUGS; i++) {
		bug b = bugs[i];
		if (b.sto > 0) {
			m3_mem[b.y][b.x] = b.col;
		}
	}
}

/*****************************************************************************\
 Food
\*****************************************************************************/

struct foodsource {
	u8  x;
	u8  y;
	u16 v;
	u8  g[SPLOTCH][SPLOTCH];
}; typedef struct foodsource food;

static const u8 FOODSHAPE[SPLOTCH][SPLOTCH] = {
	{0,0,1,1,1,0,0},
	{0,1,1,1,1,1,0},
	{1,1,1,1,1,1,1},
	{1,1,1,1,1,1,1},
	{1,1,1,1,1,1,1},
	{0,1,1,1,1,1,0},
	{0,0,1,1,1,0,0}
};

food new_food(void) {
	food f;
	f.x = qran_range(10, XMAX-10);
	f.y = qran_range(10, YMAX-10);
	f.v = 1000;
	for (int x = 0; x < SPLOTCH; x++) {
		for (int y = 0; y < SPLOTCH; y++) {
			f.g[x][y] = FOODSHAPE[x][y];
		}
	}
	return f;
}

void update_foods(food *foods) {
	for (int i = 0; i < MAXFOODS; i++) {
		food f = foods[i];
		if (f.v == 0) continue;
		for (int x = 0; x < SPLOTCH; x++) {
			for (int y = 0; y < SPLOTCH; y++) {
				int color = (f.g[x][y]) ? CLR_GRY : CLR_BLK;
				m3_mem[f.y+y+SPLOFF][f.x+x+SPLOFF] = color;
			}
		}
	}
}

/*****************************************************************************\
 Interface
 + KEY_START - starts/restarts the simulation
 + KEY_SELECT - changes from auto to manual feeding?
 + KEY_A - adds 1 food
 + KEY_B - clears all food
\*****************************************************************************/

void add_food(food *foods) {
	for (int i = 0; i < MAXFOODS; i++) {
		if (foods[i].v == 0) {
			foods[i] = new_food();
			return;
		}
	}
}

void clear_food(food *foods) {
	for (int i = 0; i < MAXFOODS; i++) {
		food f = foods[i];
		foods[i].v = 0;
		for (int x = 0; x < SPLOTCH; x++) {
			for (int y = 0; y < SPLOTCH; y++) {
				m3_mem[f.y+y+SPLOFF][f.x+x+SPLOFF] = CLR_BLK;
			}
		}
	}
}

void restart(int seed, bug *bugs, food *foods) {
	sqran(seed);
	for (int y = 0; y < YMAX; y++) {
		for (int x = 0; x < XMAX; x++) {
			m3_mem[y][x] = CLR_BLK;
		}
	}
	// new bugs
	for (int i = 0; i < MAXBUGS; i++) {
		if (i <= SPECIES) {
			bugs[i] = new_bug(255, 255, i, ENERGY_INIT);
		} else {
			bugs[i] = new_bug(0, 0, 0, 0);
		}
	}

	// new food
}

/*****************************************************************************\
 Main
\*****************************************************************************/

int main() {
	bug bugs[MAXBUGS];
	food foods[MAXFOODS];
	int seed = 0;
	int manual = 0;

	REG_DISPCNT = DCNT_MODE3 | DCNT_BG2;

	// need a splash screen that shows instructions?
	// waits for user to hit KEY_START, which seeds random with delay

	while(1) {
		seed++;
		key_poll();
		if (key_hit(KEY_START)) restart(seed, bugs, foods);
		if (key_hit(KEY_SELECT)) manual = !manual;
		if (key_hit(KEY_A)) add_food(foods);
		if (key_hit(KEY_B)) clear_food(foods);
		update_bugs(bugs);
		update_foods(foods);
	}
	return 0;
}
