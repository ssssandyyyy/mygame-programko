# mygame-programko
#include <iostream>
using namespace std;
int hrac_hp = 20;
int hrac_atk = 4;
int hrac_level = 1;
int hrac_xp = 0;
int hrac_gold = 0;
bool hrac_freeze = false;

int boss_hp = 40;
int boss_atk = 5;
int boss_turn = 0;
bool boss_shield = false;

void utok_na_bosse() {
    if (boss_shield) {
        cout << "Ledový štít absorboval útok!" << endl;
        return;
    }
    boss_hp -= hrac_atk;
    if (boss_hp < 0) boss_hp = 0;
    cout << "Zasáhl jsi Glacithorna za " << hrac_atk << " HP. Boss má nyní " << boss_hp << " HP." << endl;
}
