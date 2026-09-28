"My first GitHub repository, where I'm learning to code and saving my practice."# Pixel-the-1st

import random

MONSTERS = [
    ("Goblin", 35, 4, 8),
    ("Skeleton", 45, 5, 9),
    ("Slime", 30, 2, 6),
    ("Orc", 60, 6, 11),
]


def chance(percent):
    """Returns True percent% of the time."""
    return random.randint(1, 100) <= percent


def make_monster(fight_number):
    scale = 1 + (fight_number - 1) * 0.25

    if fight_number % 5 == 0:
        return {
            "name": "DRAGON BOSS",
            "hp": int(90 * scale),
            "low": int(8 * scale),
            "high": int(15 * scale),
        }

    name, hp, low, high = random.choice(MONSTERS)
    hp = int(hp * scale)
    low = int(low * scale)
    high = int(high * scale)

    if chance(10):
        name = "LEGENDARY " + name
        hp *= 2
        low += 3
        high += 3
    elif chance(25):
        name = "Elite " + name
        hp = int(hp * 1.5)
        high += 2

    return {"name": name, "hp": hp, "low": low, "high": high}


def player_turn(player, monster):
    """Runs the player's action. Returns True if the player is defending."""
    while True:
        choice = input("Attack, defend, or heal? ").lower()

        if choice == "attack":
            if chance(20):
                print("You missed!")
            else:
                dmg = random.randint(5, 15) + player["bonus"]
                if chance(20):
                    dmg *= 2
                    print("CRITICAL HIT!")
                monster["hp"] -= dmg
                print("You hit the", monster["name"], "for", dmg)
            return False

        elif choice == "defend":
            print("You raise your guard...")
            return True

        elif choice == "heal":
            if player["potions"] > 0:
                player["potions"] -= 1
                if chance(10):
                    player["hp"] -= 5
                    print("The potion was cursed! You lost 5 HP.")
                else:
                    amount = random.randint(8, 25)
                    player["hp"] = min(player["max_hp"], player["hp"] + amount)
                    print("You healed", amount, "HP.")
                return False
            else:
                print("No potions left!")

        else:
            print("Type attack, defend, or heal.")


def monster_turn(player, monster, defending):
    roll = random.randint(1, 100)
    if roll <= 15:
        print("The", monster["name"], "missed!")
        return

    dmg = random.randint(monster["low"], monster["high"])
    if roll >= 86:
        dmg *= 2
        print("The", monster["name"], "lands a CRITICAL HIT!")

    if defending:
        if chance(50):
            print("You blocked the attack completely!")
            return
        dmg //= 2

    player["hp"] -= dmg
    print("The", monster["name"], "hits you for", dmg)


def level_up_check(player):
    while player["xp"] >= player["level"] * 20:
        player["xp"] -= player["level"] * 20
        player["level"] += 1
        player["max_hp"] += 10
        player["hp"] = min(player["max_hp"], player["hp"] + 10)
        player["bonus"] += 1
        print("*** LEVEL UP! You are now level", player["level"], "***")


def fight(player, fight_number):
    """Returns True if the player wins."""
    monster = make_monster(fight_number)
    print("\n=== Fight", fight_number, "===")
    print("A", monster["name"], "appears!")

    while player["hp"] > 0 and monster["hp"] > 0:
        print("\nYour HP:", player["hp"], "/", player["max_hp"],
              "|", monster["name"], "HP:", monster["hp"],
              "| Potions:", player["potions"])
        defending = player_turn(player, monster)
        if monster["hp"] > 0:
            monster_turn(player, monster, defending)

    if player["hp"] <= 0:
        return False

    gold = random.randint(10, 30) + fight_number * 5
    xp = 10 + fight_number * 5
    player["gold"] += gold
    player["xp"] += xp
    print("\nYou defeated the", monster["name"], "!")
    print("You got", gold, "gold and", xp, "XP.")

    if chance(10):
        print("RARE DROP: You found a Lucky Charm! (+2 damage)")
        player["bonus"] += 2

    level_up_check(player)
    player["hp"] = min(player["max_hp"], player["hp"] + 10)
    print("You rest and recover a little HP.")
    return True


def shop(player):
    while True:
        print("\n--- SHOP --- Gold:", player["gold"])
        print("1) Potion (20 gold)")
        print("2) Sharpen sword, +2 damage (40 gold)")
        print("3) Continue to next fight")
        pick = input("Choose 1, 2 or 3: ")

        if pick == "1":
            if player["gold"] >= 20:
                player["gold"] -= 20
                player["potions"] += 1
                print("You bought a potion.")
            else:
                print("Not enough gold!")
        elif pick == "2":
            if player["gold"] >= 40:
                player["gold"] -= 40
                player["bonus"] += 2
                print("Your sword is sharper!")
            else:
                print("Not enough gold!")
        elif pick == "3":
            break
        else:
            print("Pick 1, 2 or 3.")


def play():
    player = {
        "hp": 60,
        "max_hp": 60,
        "potions": 2,
        "gold": 0,
        "xp": 0,
        "level": 1,
        "bonus": 0,
    }
    fight_number = 1

    while True:
        won = fight(player, fight_number)
        if not won:
            print("\nYou were defeated... You survived", fight_number - 1, "fights.")
            break
        fight_number += 1
        shop(player)


while True:
    play()
    again = input("\nPlay again? (yes/no) ").lower()
    if again != "yes":
        break

print("Thanks for playing!")
