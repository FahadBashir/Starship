# Starship Fontana #

This is an example C++ application using the SDL library.
It tries to be as nicely C++11 as possible but do keep in
mind that SDL is written in C and, at some stage, you have
to interface with it.

## Story ##
The evil b’Kuhn has stolen the code to Earth’s defence system.
With this code he can, at any time, defeat the entire human race.
Only one woman is brave enough to go after b’Kuhn. Will she be
Earth’s hero? Puzzle your way though the universe in the company
of Commander Fontana in **Starship Fontana**.

## Installation ##
You will have to have the SDL development libraries installed on
your system.  The easiest way to compile is to use a command-line

```bash
$ g++ -c -std=c++11 src/*.cpp
$ g++ -o starship *.o -lSDL2 -lSDL2_image
```

which will produce an executable file called "starship" in the
top-level directory.  To execute this file do the following

`$ ./starship`
 
from the top-level directory.  The game will expect to find the
`assets` directory under its current working directory.

## Code ##
the first thing i had to do to the code was allow the player to move, 
to do this i had used the SFEvent.h, and added SFEVENT_PLAYER_UP and 
SFEVENT_PLAYER_DOWN.  Next in the SFEvent.cpp i set up the new events 
with key presses, so when the user would press up arrow key the player 
would up and if the user pressed the down arrow key the player would 
move down.

After that in the SFAsset.h, i created virtual void GoNorth() and 
virtual void GoSouth(), exactly the same as the Left and Right code and 
then in the SFApp.cpp, i had added case SFEVENT_PLAYER_UP: player->GoNorth(); break, 
which makes it so that if i press the up key, it would count as GoNorth making so i 
did not have to write SFEVENT_PLAYER_UP all the time.  Next for the movement code, 
since there was couple already done for the left and right, i had just copied those 
made made the adjustments needed like changing the void SFAsset:: to GoNorth and GoSouth and 
changing the Vector2 c = *(bbox->centre) + Vector2 to (0.0f, 5.0f) for GoNorth and (0.0f, -5,0f) 
for GoSouth.

The next is making a collision between the player and the aliens in the game, for this i first 
created a collision handler between the player and the alien in the SFApp.cpp, which is: 

for(auto a : aliens) {
    if(player->CollidesWith(a)) {

      player->HandleCollision();
 }
}

Which is never similar to the code which handles the collision between the alien and projectiles, 
i had just used it as the it was the coding which i needed to do so.  Next in the SFAsset.cpp, 
i had created two auto's one called collide1 and the other called collide2 which 
were both set to equal 0, next in the movement code i had done earlier, i had set the 2 
auto's created to either 1 or 2 for each direction, check below to see which direction got what:

void SFAsset::GoWest() {
  collide2 = 1;

void SFAsset::GoEast() {
  collide2 = 2;

void SFAsset::GoNorth() {
  collide1 = 1;

void SFAsset::GoSouth() {
  collide1 = 2;

after that i had finished the collision code by adding in:

 if (SFASSET_PLAYER == type)
 {
  if (collide1 == 1)
  {
    GoSouth();
  }
  else 
  {
    GoNorth();
  }
  if (collide2 == 1)
  {
    GoEast();
  }
  else
  {
    GoWest();
  }
 }
}

This made it so that if collide1 or collide2 would be ==1, which the GoNorth and GoWest 
had the code would make then go the oppisite direction, else if they where set to GoEast 
or GoSouth, the code would be would be set to 2 and once again they would go in the 
oppisite direction.

However if anything would go north, my code would become glicthy and started to not work, which also meant that if i had fired my projectile the code would stop working, and it took me ages to figure out the solution to this, however after i had found it i made the changes:

Before ---

void SFAsset::GoNorth() {
  collide1 = 1;

After ---

void SFAsset::GoNorth() {
 if (type == SFASSET_PLAYER)
   {
     collide1 = 1;
   }

After all that since i still needed to create some walls for the game, i had first just 
went into SFAsset.h and just created a SFASSET_WALL in the enum SFASSETTYPE area, then after that
i had just created a simple box image in Inkscape and saved it in the area where all my other images were, and created i link to it which was:

case SFASSET_WALL:
    sprite = IMG_LoadTexture(sf_window->getRenderer(), "assets/wall.png");
    break;
  }

after that i had used the code which set the amount of aliens and there position and copied it and made the changes needed so it ended up like this:

 const int number_of_walls = 4;
  for(int i=0; i<number_of_walls; i++) {
    // place an wall at width/number_of_walls * i
    auto wall = make_shared<SFAsset>(SFASSET_WALL, sf_window);
    auto pos   = Point2((canvas_w/number_of_walls) * i + 100, 150.0f);
    wall->SetPosition(pos);
    walls.push_back(wall);
  }

Next i had to make 2 different collisions for the walls, the first one was with the player so when he hit it, he wouldn't go through and the way i did this was once again by using a code i had done for the player and alien collision but with the changes needed.

    for(auto w : walls) {
      if(player->CollidesWith(w)) {

       player->HandleCollision();
 }
}

After that i had to create a collision between the wall and the projectiles, for this i had used a silimar code to the aliend and projectile collision and made the necessary changes.

for(auto p : projectiles) {
    for(auto w : walls) {
      if(p->CollidesWith(w)) {
        p->HandleCollision();
        w->HandleCollision();
      }
    }
  }

And i had just simple created a render for the wall to finish it off.

  for(auto w: walls) {
   w->OnRender();
  }

The nexzt thing i had to do was create a collision between the player and the coin, which was just the projectile and alien code again with changes made to it, it turned out like:

 for(auto c : coins) {
      if(player->CollidesWith(c)) {
        player->HandleCollision();
        c->HandleCollision();
        std::cout << "You WIn" << std::endl;
      }
    }

Next i made changes to the coin render and made it so that it was set to IsAlive, i did this so that when the player would collide with it, i can set it to not alive, the code was:

 for(auto c: coins) {
    if(c->IsAlive()) {c->OnRender();}
  }

After that in SFAsset.cpp, i had just added it to the void SFAsset::HandleCollision, which turned it into 

void SFAsset::HandleCollision() {
  if(SFASSET_PROJECTILE == type || SFASSET_ALIEN == type ||SFASSET_COIN == type) {
    SetNotAlive();
  }

This is done so that the coin would be SetNotALive and dissapear, however to make the coin completely dissappear i needed to add some coding in the SFApp.cpp first, in the begining i had a problem figuring this out and i just could not understand why there was a invisible box when the coin dissapeared but after looking throught the code i fugured it out and added what needed to be added it was simply:

  list<shared_ptr<SFAsset>> tmp1;
  for(auto c : coins) {
    if(c->IsAlive()) {
      tmp1.push_back(c);
    }
  }
  coins.clear();
  coins = list<shared_ptr<SFAsset>>(tmp1);
}

This code is very similar to the alien dissapearing one and i just used it and made changes so it would work for the coin as well.

The very last thing i needed to do was add a scoring system and a end game message, which we were told to do in the terminal and just use cout for.  For the scoring system, i first created a auto score which equaled 0 and then in the alien and projectile code i made it so that when an alien would be destroyed score would equal score + 10, the coding looked like:

 for(auto p : projectiles) {
    for(auto a : aliens) {
      if(p->CollidesWith(a)) {
        score = score + 10;
        std::cout << score << std::endl;
        p->HandleCollision();
        a->HandleCollision();
      }
    }
  }

and for the end game message i simply just wanted it to appear when the player and coin collision took place which meant that when the player collected the coin the game would end.  So i just made some changes to the collision could which i created for the player and coin and made some changes.  The code looking like:

 for(auto c : coins) {
      if(player->CollidesWith(c)) {
        player->HandleCollision();
        c->HandleCollision();
        std::cout << "You WIn" << std::endl;
      }
    }

However the game does not truly end, only the message in the terminal says it does.

## Credits ##
The sprites in this game come directly from 
[SpriteLib](http://www.widgetworx.com/widgetworx/portfolio/spritelib.html) and are used
under the terms of the [CPL 1.0](http://opensource.org/licenses/cpl1.0.php).
