#include <iostream>
#include <vector>
#include <cstdlib>
#include <ctime>
using namespace std;

struct Skill {
    int power;
    int manaUse;
    int lifeUse;
};

struct Hero {
    float health;
    float energy;
    float maxHealth;
    float maxEnergy;
    int money;
    int experience;
    vector<Skill> arsenal;
};

int getRandom(int low, int high) {
    return rand() % (high - low + 1) + low;
}

void upgradeShop(Hero &champ) {
    while (true) {
        cout << "\n--- Recovery Station ---\nGold: " << champ.money << "\n";
        cout << "[1] Restore HP (10g)\n[2] Boost Max HP +10 (15g)\n";
        cout << "[3] Restore MP (5g)\n[4] Boost Max MP +20 (15g)\n";
        cout << "[5] Experience Use\n[6] Exit\n";
        int pick;
        cin >> pick;

        switch (pick) {
            case 1:
                if (champ.money >= 10) {
                    champ.health = champ.maxHealth;
                    champ.money -= 10;
                    cout << "Full HP: " << champ.health << "\n";
                } else cout << "Need more gold!\n";
                break;
            case 2:
                if (champ.money >= 15) {
                    champ.maxHealth += 10;
                    champ.money -= 15;
                    cout << "Max HP now: " << champ.maxHealth << "\n";
                } else cout << "Not enough gold.\n";
                break;
            case 3:
                if (champ.money >= 5) {
                    champ.energy = champ.maxEnergy;
                    champ.money -= 5;
                    cout << "MP Restored: " << champ.energy << "\n";
                } else cout << "Gold missing.\n";
                break;
            case 4:
                if (champ.money >= 15) {
                    champ.maxEnergy += 20;
                    champ.money -= 15;
                    cout << "Max MP now: " << champ.maxEnergy << "\n";
                } else cout << "Gold insufficient.\n";
                break;
            case 5:
                cout << "XP: " << champ.experience << "/100\n";
                if (champ.experience >= 100) {
                    champ.experience = 0;
                    champ.maxHealth += 10;
                    champ.maxEnergy += 10;
                    champ.health = champ.maxHealth;
                    champ.energy = champ.maxEnergy;
                    for (Skill &s : champ.arsenal) s.power += 5;
                    cout << "LEVEL UP!\n";
                }
                break;
            case 6:
                return;
            default:
                cout << "Try again.\n";
        }
    }
}

void fightEnemies(Hero &player, vector<float> enemies, string name, float multiplier) {
    if (name == "Village Encounter") {
        cout << "\nYou arrive at a village. Time to rest and recover.\n";
        upgradeShop(player);
        return;
    }

    while (true) {
        cout << "\nBattle against: " << name << "\n";
        for (int i = 0; i < enemies.size(); ++i)
            cout << "Enemy " << i + 1 << ": " << (enemies[i] > 0 ? to_string(enemies[i]) + " HP" : "Defeated") << "\n";

        int target;
        cout << "Choose target (1-" << enemies.size() << "): ";
        cin >> target;
        if (target < 1 || target > enemies.size() || enemies[target - 1] <= 0) {
            cout << "Invalid choice.\n";
            continue;
        }

        int skillIndex;
        cout << "Choose skill (1-" << player.arsenal.size() << "): ";
        cin >> skillIndex;

        if (skillIndex < 1 || skillIndex > player.arsenal.size()) {
            cout << "Invalid skill.\n";
            continue;
        }

        Skill chosen = player.arsenal[skillIndex - 1];
        if (player.energy < chosen.manaUse || player.health <= chosen.lifeUse) {
            cout << "Insufficient resources.\n";
            continue;
        }

        player.energy -= chosen.manaUse;
        player.health -= chosen.lifeUse;
        enemies[target - 1] -= chosen.power;

        bool allDefeated = true;
        for (float hp : enemies) {
            if (hp > 0) allDefeated = false;
        }
        if (allDefeated) {
            cout << "\nAll enemies defeated!\n";
            player.experience += 20;
            player.money += getRandom(5, 15);
            break;
        }

        for (int i = 0; i < enemies.size(); ++i) {
            if (enemies[i] > 0) {
                int dmg = getRandom(5, 10);
                player.health -= dmg * multiplier;
                cout << "Enemy " << (i + 1) << " attacks for " << dmg * multiplier << " damage.\n";
            }
        }

        if (player.health <= 0) {
            cout << "\nYou were defeated in battle...\n";
            exit(0);
        }
    }
}

struct GlacithornBoss {
    float hp = 120;
    bool shieldActive = false;
    int turnCount = 0;
    bool freezePlayer = false;

    void activateShield() {
        shieldActive = true;
        cout << ">>> Glacithorn summons an ICE SHIELD! All damage blocked this turn.\n";
    }

    void absorbDamage(int dmg) {
        if (shieldActive) {
            cout << ">>> Glacithorn's shield absorbed the attack!\n";
            shieldActive = false;
            return;
        }

        hp -= dmg;
        cout << "Glacithorn takes " << dmg << " damage! Remaining HP: " << hp << "\n";
    }

    void endOfTurnCheck() {
        if (shieldActive) {
            hp += 5;
            cout << ">>> Shield was not broken. Glacithorn regenerates 5 HP! New HP: " << hp << "\n";
            shieldActive = false;
        }
    }

    void performAttack(Hero &player) {
        turnCount++;

        if (turnCount % 6 == 0) {
            freezePlayer = true;
            cout << "\n>>> Glacithorn uses FROST STRIKE!\n";
            cout << "\"Glacithorn ude\u0159il p\u0159\u00edmo do srdce zimou v\u011bk\u016f!\"\n";
            return;
        }

        int baseDmg = getRandom(8, 16);
        player.health -= baseDmg;
        cout << "Glacithorn hits you for " << baseDmg << " damage. Your HP: " << player.health << "\n";

        if (turnCount % 3 == 0) {
            activateShield();
        }
    }

    bool shouldPlayerFailAction() {
        return (turnCount % 2 == 1 && getRandom(1, 100) <= 25);
    }

    bool isPlayerFrozen() {
        return freezePlayer;
    }

    void resetFreeze() {
        freezePlayer = false;
    }

    bool isDefeated() {
        return hp <= 0;
    }
};

int main() {
    srand(time(0));

    Hero player;
    int type;
    cout << "Choose your adventurer:\n1) Sellsword (100 HP, 80 MP)\n2) Fighter (120 HP, 40 MP)\n";
    cin >> type;

    if (type == 1) {
        player = {100, 80, 100, 80, 10, 0, {{10, 0, 0}, {20, 10, 0}, {30, 0, 10}}};
    } else {
        player = {120, 40, 120, 40, 10, 0, {{15, 0, 0}, {20, 10, 0}, {30, 0, 5}}};
    }

    vector<pair<string, vector<float>>> waveSequence = {
        {"Village Encounter", {}},
        {"Monster", {25}},
        {"Monster", {30}},
        {"Double Monster", {20, 25}},
        {"Mini Boss", {60}},
        {"Village Encounter", {}},
        {"Monster", {25}},
        {"Double Monster", {20, 20}},
        {"Double Monster", {30, 30}},
        {"Mini Boss", {70}},
        {"Village Encounter", {}},
        {"Double Monster", {35, 35}},
        {"Double Monster", {40, 40}},
        {"Triple Monster", {30, 30, 30}},
        {"Village Encounter", {}} // Last village before final boss
    };

    vector<float> multipliers = {1, 1, 1.2, 1.3, 1.5, 1, 1, 1.3, 1.4, 1.6, 1, 1.4, 1.5, 1.6, 1};

    for (int i = 0; i < waveSequence.size(); ++i) {
        fightEnemies(player, waveSequence[i].second, waveSequence[i].first, multipliers[i]);
    }

    GlacithornBoss glacithorn;
    cout << "\n>>> FINAL BATTLE: GLACITHORN THE ICE GIANT <<<\n";

    while (player.health > 0 && !glacithorn.isDefeated()) {
        cout << "\nHP: " << player.health << " | MP: " << player.energy << "\n";
        cout << "Glacithorn HP: " << glacithorn.hp << "\n";

        if (glacithorn.isPlayerFrozen()) {
            cout << "\nYou're frozen solid and miss your turn!\n";
            glacithorn.resetFreeze();
        } else {
            if (glacithorn.shouldPlayerFailAction()) {
                cout << "\nTime slows... Your action fails due to Glacithorn's icy aura!\n";
            } else {
                int atkChoice;
                cout << "Choose attack (1-" << player.arsenal.size() << "): ";
                cin >> atkChoice;

                if (atkChoice < 1 || atkChoice > player.arsenal.size()) continue;

                Skill chosen = player.arsenal[atkChoice - 1];

                if (player.energy < chosen.manaUse || player.health <= chosen.lifeUse) {
                    cout << "Not enough MP or HP to perform this attack.\n";
                    continue;
                }

                player.energy -= chosen.manaUse;
                player.health -= chosen.lifeUse;

                glacithorn.absorbDamage(chosen.power);
            }
        }

        if (glacithorn.isDefeated()) {
            cout << "\n>>> You shattered the Ice Giant! Glacithorn has fallen.\n";
            break;
        }

        glacithorn.performAttack(player);
        glacithorn.endOfTurnCheck();

        if (player.health <= 0) {
            cout << "\n>>> Your soul is frozen... You were defeated by Glacithorn.\n";
            break;
        }
    }

    return 0;
}
