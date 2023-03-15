# ad
import javax.swing.*;
import java.awt.event.*;
import java.awt.*;

public class TicTacToeMain extends JFrame implements ActionListener {

 private JButton [][]buttons = new JButton[3][3];
 private JButton playButton = new JButton("Play");
 private JLabel statusLabel = new JLabel("");
 private TicTacToeAI game = null;
 private int human = 0;
 private int computer = 0;
 private boolean isPlay = false;
 private String []chars=new String[]{"","X","O"};

 private void setStatus(String s) {
  statusLabel.setText(s);
 }

 private void setButtonsEnabled(boolean enabled) {
  for(int i=0;i<3;i++)
   for(int j=0;j<3;j++) {
    buttons[i][j].setEnabled(enabled);
    if(enabled) buttons[i][j].setText(" ");
   }
 }

 public TicTacToeMain() {

  setTitle("Tic Tac Toe");
  setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
  setResizable(false);

  JPanel centerPanel = new JPanel(new GridLayout(3,3));
  Font font = new Font("Arial",Font.BOLD, 32);
  for(int i=0;i<3;i++)
   for(int j=0;j<3;j++) {
    buttons[i][j] = new JButton(" ");
    buttons[i][j].setFont(font);
    buttons[i][j].addActionListener(this);
    buttons[i][j].setFocusable(false);
    centerPanel.add(buttons[i][j]);
   }

  playButton.addActionListener(this);

  JPanel northPanel = new JPanel();
  northPanel.add(statusLabel);

  JPanel southPanel = new JPanel();
  southPanel.add(playButton);

  setStatus("Click 'Play' To Start");
  setButtonsEnabled(false);

  add(northPanel,"North");
  add(centerPanel,"Center");
  add(southPanel,"South");

  setSize(300,300);

  // I'm lazy to implement the correct way
  setLocationRelativeTo(null);
 }

 public static void main(String []args) {
  new TicTacToeMain().setVisible(true);
 }

 private void computerTurn() {
  if(!isPlay) return;

  int []pos = game.nextMove(computer);
  if(pos!=null) {
   int i = pos[0];
   int j = pos[1];
   buttons[i][j].setText(chars[computer]);
   game.setBoardValue(i,j,computer);
  }

  checkState();
 }

 private void gameOver(String s) {
  setStatus(s);
  setButtonsEnabled(false);
  isPlay = false;
 }

 private void checkState() {
  if(game.isWin(human)) {
   gameOver("Congratulations, You've Won!");
  }
  if(game.isWin(computer)) {
   gameOver("Sorry, You Lose!");
  }
  if(game.nextMove(human)==null && game.nextMove(computer)==null) {
   gameOver("Draw, Click 'Play' For Rematch!");
  }
 }

 private void click(int i,int j) {
  if(game.getBoardValue(i,j)==TicTacToeAI.EMPTY) {
   buttons[i][j].setText(chars[human]);
   game.setBoardValue(i,j,human);
   checkState();
   computerTurn();
  }
 }

 public void actionPerformed(ActionEvent event) {
  if(event.getSource()==playButton) {
   play();
  }else {
   for(int i=0;i<3;i++)
    for(int j=0;j<3;j++)
     if(event.getSource()==buttons[i][j])
      click(i,j);
  }
 }

 private void play() {
  game = new TicTacToeAI();
  human = TicTacToeAI.ONE;
  computer = TicTacToeAI.TWO;
  setStatus("Your Turn");
  setButtonsEnabled(true);
  isPlay = true;
 }
}

-------------------------------------------------------------------------------------------------------------------------------------

public class TicTacToeAI {

 /* the board */
 private int board[][];
 /* empty */
 public static final int EMPTY = 0;
 /* player one */
 public static final int ONE = 1;
 /* player two */
    public static final int TWO = 2;

 public TicTacToeAI() {
  board = new int[3][3];
 }

 /* get the board value for position (i,j) */
 public int getBoardValue(int i,int j) {
  if(i < 0 || i >= 3) return EMPTY;
  if(j < 0 || j >= 3) return EMPTY;
  return board[i][j];
    }

 /* set the board value for position (i,j) */
 public void setBoardValue(int i,int j,int token) {
  if(i < 0 || i >= 3) return;
  if(j < 0 || j >= 3) return;
  board[i][j] = token;
    }

 /* calculate the winning move for current token */
 public int []nextWinningMove(int token) {

  for(int i=0;i<3;i++)
   for(int j=0;j<3;j++)
    if(getBoardValue(i, j)==EMPTY) {
     board[i][j] = token;
     boolean win = isWin(token);
     board[i][j] = EMPTY;
     if(win) return new int[]{i,j};
    }

  return null;
    }

    public int inverse(int token) {
  return token==ONE ? TWO : ONE;
 }

    /* calculate the best move for current token */
    public int []nextMove(int token) {

        /* lucky position in the center of board*/
        if(getBoardValue(1, 1)==EMPTY) return new int[]{1,1};

        /* if we can move on the next turn */
        int winMove[] = nextWinningMove(token);
        if(winMove!=null) return winMove;

        /* choose the move that prevent enemy to win */
        for(int i=0;i<3;i++)
            for(int j=0;j<3;j++)
                if(getBoardValue(i, j)==EMPTY)
                {
                    board[i][j] = token;
              boolean ok = nextWinningMove(inverse(token)) == null;
                    board[i][j] = EMPTY;
                    if(ok) return new int[]{i,j};
                }

        /* choose available move */
        for(int i=0;i<3;i++)
            for(int j=0;j<3;j++)
                if(getBoardValue(i, j)==EMPTY)
                    return new int[]{i,j};

        /* no move is available */
        return null;
    }

 /* determine if current token is win or not win */
 public boolean isWin(int token) {
  final int DI[]={-1,0,1,1};
  final int DJ[]={1,1,1,0};
  for(int i=0;i<3;i++)
   for(int j=0;j<3;j++) {

    /* we skip if the token in position(i,j) not equal current token */
    if(getBoardValue(i, j)!=token) continue;

    for(int k=0;k<4;k++) {
     int ctr = 0;
                 while(getBoardValue(i+DI[k]*ctr, j+DJ[k]*ctr)==token) ctr++;

     if(ctr==3) return true;
    }
  }
  return false;
    }
} 
