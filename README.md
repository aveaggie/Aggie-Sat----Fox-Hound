#include "std_lib_facilities_3.h"
#include "Simple_window.h"
#include "Graph.h"
#include "pieces.h"
#include <map>


using namespace Graph_lib;
using namespace game_play;


Lines_window winstartwindow(Point(100,100),630,560,"Game");
// Creates a vector of strings holding all of team names for the SEC
Vector<string>SECDefence = {"Alabama","Arkansas","Auburn","Carolina","Florida","Georgia","Kentucky","LSU","Michigan","Ole Miss","Missouri","Tennesee","Vanderbilt"};
In_box Player_name(Point(336,528),100,20,"Name:");
map<int,string> Highscores;
//-------------------------------------STRUCTS-----------------------------------------
struct My_Button : public Button
{
    int row, column; //each My_Button knows its position  
    My_Button(Point xy, int w, int h, const string& label, Callback cb, int r, int c)
                            : Button(xy,w,h,label,cb),row(r), column(c){}
    void attach(Graph_lib::Window& win)
	{
        pw = new Fl_Button(loc.x, loc.y, width, height, label.c_str());
        pw->callback(reinterpret_cast<Fl_Callback*>(do_it), this);
        own = &win;
	}
    void set_fill_color(Color c){pw->color(Fl_Color(c.as_int()));}
    void redraw(){own->redraw();}
	
};


Lines_window::Lines_window(Point xy, int w, int h, const string& title)
    :Window(xy,w,h,title),
	quit_button(Point(560,0), 70, 35, "Quit", cb_quit)
{

}  

//--------------------------------CALLBACKS-----------------------------------------------

void Lines_window::cb(Address, Address pb)  
{  
    int r = reference_to<My_Button>(pb).row; 
    int c =	reference_to<My_Button>(pb).column;

	if((turn_count%2 == 0)&&(abs(qb.old_row-r) == 1)&&((abs(qb.old_column-c) == 1)||(turn_count == 0))&&(has_hound(r,c) == false))
	{
		qb.move(r,c);
		off.move(100,0);
		reference_to<My_Button>(pb).redraw();
		def.move(-100,0);
		qb.old_row=r;
		qb.old_column=c;
		++turn_count;
	}
	else if((turn_count%2 == 1)&&((hound_count == 0)||(has_hound(r,c) == true)))
	{
		if(def1.row() == r && def1.col() == c)
			hound_count = 1;
		else if(def2.row() == r && def2.col() == c)
			hound_count = 2;
		else if(def3.row() == r && def3.col() == c)
			hound_count = 3;
		else if(def4.row() == r && def4.col() == c)	
			hound_count = 4;
	}
	else if((hound_count == 1)&&(def1.old_row-r == -1)&&(abs(def1.old_column-c) == 1)&&(has_fox(r,c) == false)&&(has_hound(r,c) == false)){
		move_to_button(def1, r, c);
		reference_to<My_Button>(pb).redraw();
		def.move(100,0);}
	else if((hound_count == 2)&&(def2.old_row-r == -1)&&(abs(def2.old_column-c) == 1)&&(has_fox(r,c) == false)&&(has_hound(r,c) == false)){
		move_to_button(def2, r, c);
		reference_to<My_Button>(pb).redraw();
		def.move(100,0);}
	else if((hound_count == 3)&&(def3.old_row-r == -1)&&(abs(def3.old_column-c) == 1)&&(has_fox(r,c) == false)&&(has_hound(r,c) == false)){
		move_to_button(def3, r, c);
		reference_to<My_Button>(pb).redraw();
		def.move(100,0);}
	else if((hound_count == 4)&&(def4.old_row-r == -1)&&(abs(def4.old_column-c) == 1)&&(has_fox(r,c) == false)&&(has_hound(r,c) == false)){
		move_to_button(def4, r, c);
		reference_to<My_Button>(pb).redraw();
		def.move(100,0);}
	cout << turn_count/2 << "\n";
} 

void Lines_window::cb_cpu_hound(Address, Address pb)  
{  
    int r = reference_to<My_Button>(pb).row; 
    int c =	reference_to<My_Button>(pb).column;
	set_dist();
	if((abs(qb.old_row-r) == 1)&&((abs(qb.old_column-c) == 1)||(turn_count == 0))&&(has_hound(r,c) == false))
	{
		off.move(100,0);
		qb.move(r,c);
		qb.move(r,c);
		reference_to<My_Button>(pb).redraw();
		off.move(-100,0);
		qb.old_row=r;
		qb.old_column=c;
		switch(farthest_hound())
		{
		case 1:
			best_move(def1,def1.row(),def1.col());
			reference_to<My_Button>(pb).redraw();
			break;
		case 2:
			best_move(def2,def2.row(),def2.col());
			reference_to<My_Button>(pb).redraw();
			break;
		case 3:
			best_move(def3,def3.row(),def3.col());
			reference_to<My_Button>(pb).redraw();
			break;
		case 4:
			best_move(def4,def4.row(),def4.col());
			reference_to<My_Button>(pb).redraw();
			break;
		}
		turn_count+=2;
	}
	cout << turn_count/2 << "\n";
} 
void Lines_window::cpu_fox(Address, Address pb)
{
	int r = reference_to<My_Button>(pb).row; 
    int c =	reference_to<My_Button>(pb).column;
	if((hound_count == 0)||(has_hound(r,c) == true))
	{
		if(def1.row() == r && def1.col() == c)
			hound_count = 1;
		else if(def2.row() == r && def2.col() == c)
			hound_count = 2;
		else if(def3.row() == r && def3.col() == c)
			hound_count = 3;
		else if(def4.row() == r && def4.col() == c)	
			hound_count = 4;
	}
	else if((hound_count == 1)&&(def1.old_row-r == -1)&&(abs(def1.old_column-c) == 1)&&(has_fox(r,c) == false)&&(has_hound(r,c) == false)){
		move_to_button(def1, r, c);
		cpu_fox_move();
		reference_to<My_Button>(pb).redraw();}
	else if((hound_count == 2)&&(def2.old_row-r == -1)&&(abs(def2.old_column-c) == 1)&&(has_fox(r,c) == false)&&(has_hound(r,c) == false)){
		move_to_button(def2, r, c);
		cpu_fox_move();
		reference_to<My_Button>(pb).redraw();}
	else if((hound_count == 3)&&(def3.old_row-r == -1)&&(abs(def3.old_column-c) == 1)&&(has_fox(r,c) == false)&&(has_hound(r,c) == false)){
		move_to_button(def3, r, c);
		cpu_fox_move();
		reference_to<My_Button>(pb).redraw();}
	else if((hound_count == 4)&&(def4.old_row-r == -1)&&(abs(def4.old_column-c) == 1)&&(has_fox(r,c) == false)&&(has_hound(r,c) == false)){
		move_to_button(def4, r, c);
		cpu_fox_move();
		reference_to<My_Button>(pb).redraw();}
	if(qb.row() == 0)
	{
		
		fox_win = true;
	}
	cout << turn_count/2 << "\n";
}
void Lines_window::Instructions(Address, Address pb)// Callback for the instructions button opens a pdf
{	
	system("xpdf Instructions.pdf");
}
void Lines_window::playcb(Address, Address pb) //Closes the first window and runs the play function
{
	winstartwindow.hide();
	reference_to<Lines_window>(pb).Play();
}

void Lines_window::One_Player(Address, Address pb)//Call back for the oneplayer button and closes the previous
{												  // window and runs the Choice funcion. Increments condition.
	condition++;
	reference_to<Lines_window>(pb).quit();
	reference_to<Lines_window>(pb).Choice();
}
void Lines_window::One_Player_Defence(Address, Address pb)//Closes the previous window and runs choice function
{														  //increments by two to run a different call back
	condition += 2;
	reference_to<Lines_window>(pb).quit();	
	reference_to<Lines_window>(pb).Choice();
}
void Lines_window::Two_Player(Address, Address pb)// Closes previous window runs choice function again
{											      
	reference_to<Lines_window>(pb).quit();
	reference_to<Lines_window>(pb).Choice();
}
void Lines_window::cbb(Address, Address pb)// Callback that does nothing
{

}
void Lines_window::Defencecb(Address, Address pb) // 12 callbacks that close the previous window and runs
{												  // the game function, and depending on what button is clicked
	reference_to<Lines_window>(pb).quit();		  // and increments differently.
	reference_to<Lines_window>(pb).Game();
}
void Lines_window::Defencecb2(Address, Address pb)
{
	trial++;
	reference_to<Lines_window>(pb).quit();
	reference_to<Lines_window>(pb).Game();
}
void Lines_window::Defencecb3(Address, Address pb)
{
	trial+=2;
	reference_to<Lines_window>(pb).quit();
	reference_to<Lines_window>(pb).Game();
}
void Lines_window::Defencecb4(Address, Address pb)
{
	trial+=3;
	reference_to<Lines_window>(pb).quit();
	reference_to<Lines_window>(pb).Game();
}
void Lines_window::Defencecb5(Address, Address pb)
{
	trial+=4;
	reference_to<Lines_window>(pb).quit();
	reference_to<Lines_window>(pb).Game();
}
void Lines_window::Defencecb6(Address, Address pb)
{
	trial+=5;
	reference_to<Lines_window>(pb).quit();
	reference_to<Lines_window>(pb).Game();
}
void Lines_window::Defencecb7(Address, Address pb)
{
	trial+=6;
	reference_to<Lines_window>(pb).quit();
	reference_to<Lines_window>(pb).Game();
}
void Lines_window::Defencecb8(Address, Address pb)
{
	trial+=7;
	reference_to<Lines_window>(pb).quit();
	reference_to<Lines_window>(pb).Game();
}
void Lines_window::Defencecb9(Address, Address pb)
{
	trial+=8;
	reference_to<Lines_window>(pb).quit();
	reference_to<Lines_window>(pb).Game();
}
void Lines_window::Defencecb0(Address, Address pb)
{
	trial+=9;
	reference_to<Lines_window>(pb).quit();
	reference_to<Lines_window>(pb).Game();
}
void Lines_window::Defencecb11(Address, Address pb)
{
	trial+=10;
	reference_to<Lines_window>(pb).quit();
	reference_to<Lines_window>(pb).Game();
}
void Lines_window::Defencecb12(Address, Address pb)
{
	trial+=11;
	reference_to<Lines_window>(pb).quit();
	reference_to<Lines_window>(pb).Game();
}
void Lines_window::Defencecb13(Address, Address pb)
{
	trial+=12;
	reference_to<Lines_window>(pb).quit();
	reference_to<Lines_window>(pb).Game();
}
void Lines_window::repeatyescb(Address, Address pb) //Unsucessful attempt to create a repeat callback
{
	trial *= 0;
	condition = 1;
	reference_to<Lines_window>(pb).quit();
	reference_to<Lines_window>(pb).Play();
}
void Lines_window::highscorecb(Address, Address pb)
{

}
void Lines_window::cb_quit(Address, Address pb) // A quit callback for a quit button
{
	reference_to<Lines_window>(pb).quit();
	reference_to<Lines_window>(pb).High_Score();
}
void Lines_window::cb_quit2(Address, Address pb) // A quit callback for a quit button
{
	reference_to<Lines_window>(pb).quit();
}


//----------------------------------Functions-----------------------------------------------
void Lines_window::checkerboard() //Function that when it is run it creates a checkerboard of alternating buttons
{								  //black buttons are the only ones that are clickable.	
	  vector<vector<My_Button*> >board(8,vector<My_Button*>(8));
 
  for(int i = 0; i < 8; ++i)
  {
        for(int j = 0; j < 8; ++j)
		{
            if((i + j) % 2)
			{
				board[i][j] = new My_Button(Point(70*j, 70*i),70, 70, "", Lines_window::cbb,i, j);
				attach(*board[i][j]); 

				if((i + j) % 2)
				{
					board[i][j]->set_fill_color(fl_rgb_color(100,100,100));
				}
			}

			if(!((i + j) % 2))
			{	
				if(condition == 1)
				{
					board[i][j] = new My_Button(Point(70*j, 70*i),70, 70, "",Lines_window::cb,i, j);
				}
				else if(condition == 2)
				{
					board[i][j] = new My_Button(Point(70*j, 70*i),70, 70, "",Lines_window::cb_cpu_hound,i, j);
				}
				else if(condition == 3)
				{
					board[i][j] = new My_Button(Point(70*j, 70*i),70, 70, "",Lines_window::cpu_fox,i, j);
				}
				attach(*board[i][j]); 
				board[i][j]->set_fill_color(fl_rgb_color(60,60,60));
			}   
        }
   }
}  	
int Lines_window::Play() //Play fuctions that runs only when it is called by a button creates a window with 3
{						 //buttons which run the One_Player, Two_Player, and One_Player_defence callbacks.
		Lines_window winplay(Point(100,100),630,560,"Game");
		winplay.attach(gameimg);
		Button Oneplayer_Button(Point(43,227),545,83,"One Player(Quarterback)",Lines_window::One_Player);
		Button Twoplayer_Button(Point(44,327),543,79,"Two Player",Lines_window::Two_Player);
		Button Oneplayer2_Button(Point(44,424),543,80,"One Player(Defence)",Lines_window::One_Player_Defence);
		winplay.attach(Oneplayer_Button);
		winplay.attach(Twoplayer_Button);
		winplay.attach(Oneplayer2_Button);
		return gui_main();
}
int Lines_window::Choice() //Creates a window that attaches 12 buttons for the SEC teams, and has an if statement
{						   //that moves the banners on the side of the checkerboard window.	
	Lines_window winchoice(Point(100,100),630,560,"Game2");	
	winchoice.attach(teamsimg);
	if(condition == 3){
		off.move(100,0);
		def.move(-100,0);}
	Button Oneplayer2_Button(Point(9,385),71,71,"Alabama",Lines_window::Defencecb);
	Button Oneplayer3_Button(Point(106,384),71,71,"Arkansas",Lines_window::Defencecb2);
	Button Oneplayer4_Button(Point(11,293),71,71,"Auburn",Lines_window::Defencecb3);
	Button Oneplayer5_Button(Point(199,479),71,71,"Carolina",Lines_window::Defencecb4);
	Button Oneplayer6_Button(Point(464,480),71,71,"Florida",Lines_window::Defencecb5);
	Button Oneplayer7_Button(Point(550,475),71,71,"Georgia",Lines_window::Defencecb6);
	Button Oneplayer8_Button(Point(379,480),71,71,"Kentucky",Lines_window::Defencecb7);
	Button Oneplayer9_Button(Point(292,480),71,71,"LSU",Lines_window::Defencecb8);
	Button Oneplayer0_Button(Point(11,479),71,71,"Michigan",Lines_window::Defencecb9);
	Button Oneplayer11_Button(Point(108,481),71,71,"Ole Miss",Lines_window::Defencecb0);
	Button Oneplayer12_Button(Point(547,292),71,71,"Mizzouri",Lines_window::Defencecb11);
	Button Oneplayer13_Button(Point(552,382),71,71,"Tennesee",Lines_window::Defencecb12);
	Button Oneplayer14_Button(Point(466,387),71,71,"Vanderbilt",Lines_window::Defencecb13);
	winchoice.attach(Oneplayer2_Button);
	winchoice.attach(Oneplayer3_Button);
	winchoice.attach(Oneplayer4_Button);
	winchoice.attach(Oneplayer5_Button);
	winchoice.attach(Oneplayer6_Button);
	winchoice.attach(Oneplayer7_Button);
	winchoice.attach(Oneplayer8_Button);
	winchoice.attach(Oneplayer9_Button);
	winchoice.attach(Oneplayer0_Button);
	winchoice.attach(Oneplayer11_Button);
	winchoice.attach(Oneplayer12_Button);
	winchoice.attach(Oneplayer13_Button);
	winchoice.attach(Oneplayer14_Button);
	return gui_main();
}

int Lines_window::Game()//Game function that creates a window that calls the checkerboard function which draws
{						//a checkerboard and on the board is 5 images. One tamu and whatever SEC team you choose.
  Lines_window wingame(Point(100,100),630,560,"Game2");		 
  Button quit_button(Point(560,0), 70, 35, "Quit", cb_quit);
  wingame.checkerboard();
  wingame.attach(backg);
  wingame.attach(quit_button);
  do{
  Image tamu(Point(qb.x_coor()-32,qb.y_coor()-32),"TAMU.gif");
  Image alabama1(Point(def1.x_coor()-32,def1.y_coor()-32), SECDefence[trial] + ".gif");
  Image alabama2(Point(def2.x_coor()-32,def2.y_coor()-32), SECDefence[trial] + ".gif");
  Image alabama3(Point(def3.x_coor()-32,def3.y_coor()-32), SECDefence[trial] + ".gif");
  Image alabama4(Point(def4.x_coor()-32,def4.y_coor()-32), SECDefence[trial] + ".gif");
  fox_is_surrounded();
  wingame.attach(tamu);
  wingame.attach(off);
  wingame.attach(def);
  wingame.attach(alabama1);
  wingame.attach(alabama2);
  wingame.attach(alabama3);
  wingame.attach(alabama4); 
  
  Fl::wait(10);
  }while((qb.old_row != 0)&&(wingame.pressed == false)&&(hound_victory() == false)&&(fox_victory() == false));
 
  if(hound_victory() == true)	// These if statements calculate the score and attach victory banners depending
  {								// on who wins.
	  fox_score = (turn_count/2)*2;
	  hound_score = 100 - fox_score;
	  wingame.attach(secvic);
	  
  }
  else
  {
	  fox_score = (turn_count/2)*3;
	  hound_score = 100 - fox_score;
	  wingame.attach(tamuvic);
  }
  cout << "Score: " << fox_score << "\n";
  if(hound_score > fox_score)
  {
	  score = hound_score;
  }
  else
	  score = fox_score;

  return gui_main();
}


int Start_window()// Attaches 2 buttons and an inbox to input your name.
{ //First Window where player chooses what to play as.
	Button play_Button(Point(270,295),118,76,"Play",Lines_window::playcb);
	Button Rules_Button(Point(169,407),323,64,"Rules & Instructions",Lines_window::cb_quit2);
	{
		
		winstartwindow.attach(coverimg);
		winstartwindow.attach(Player_name);
		winstartwindow.attach(play_Button);
		winstartwindow.attach(Rules_Button);
		return gui_main();
	} 
}
int Lines_window::High_Score()
{
	Lines_window winhighscore(Point(100,100),630,560,"Highscores");
	Button newquitbutton(Point(600,0),30,30,"Quit",Lines_window::cb_quit2);
	Circle c1(Point(100,200),50);
	string asdf = Highscores[score];
	Text high(Point (200,200),asdf);
	winhighscore.attach(high);
	winhighscore.attach(c1);
	winhighscore.attach(newquitbutton);
	return gui_main();

}
int main() //main function only runs the start_window() function
{
  Start_window();
  Lines_window winhighscore(Point(100,100),630,560,"Highscores");
  string ifile = "Highscores.txt";
  string testname;
  int testscore;
  inputname = Player_name.get_string();
  cout << "NAME: " << inputname << "\n";

  ifstream ist(ifile.c_str());
  ofstream ost(ifile.c_str(), ios::out | ios::app);
  while(!ist.eof())
  {
  ist >> testscore >> testname;
  Highscores.insert(std::pair<int,string>(testscore, testname));
  }
  Highscores[score] = inputname;
  Highscores[67] = "Miguel";
  Highscores[2] = "Sergio"; 
  ost << score << " " << inputname << "\n";
  int place = Highscores.size();
  for(map<int,string>::iterator ii = Highscores.begin(); ii!=Highscores.end(); ++ii)
  {
	  cout <<place<<": "<< (*ii).first << "-" << (*ii).second << endl;
	  place--;
  }
}
