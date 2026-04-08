import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.net.URI;

public class PongGame extends JPanel implements ActionListener, KeyListener {

    // ---------- Window ----------
    static final int WIDTH = 800, HEIGHT = 600;

    // ---------- Ball ----------
    int ballX = WIDTH/2, ballY = HEIGHT/2;
    int ballXV = -4, ballYV = 4; // slightly faster
    final int BALL_SIZE = 15;

    // ---------- Paddles ----------
    int p1Y = HEIGHT/2 - 50; // AI
    int p2Y = HEIGHT/2 - 50; // Player
    final int PADDLE_W = 10, PADDLE_H = 100;

    // ---------- Score ----------
    int p1Score=0, p2Score=0;
    final int WIN_SCORE = 5;

    Timer timer;
    JFrame pongFrame;
    JButton playAgainButton;

    // ---------- AI ----------
    int aiSpeed = 5;           // faster
    double aiReaction = 0.8;   // 80% chance AI reacts each frame

    // ---------- Player movement ----------
    boolean upPressed=false;
    boolean downPressed=false;

    // ---------- Constructor ----------
    public PongGame(JFrame frame, String difficulty){
        this.pongFrame = frame;
        setDifficulty(difficulty);

        timer = new Timer(1000/60,this); // 60 FPS
        timer.start();

        addKeyListener(this);
        setFocusable(true);
        requestFocusInWindow();
    }

    private void setDifficulty(String difficulty){
        switch(difficulty.toLowerCase()){
            case "easy": aiSpeed=3; aiReaction=0.6; break;
            case "medium": aiSpeed=5; aiReaction=0.8; break;
            case "hard": aiSpeed=7; aiReaction=0.95; break;
            default: aiSpeed=5; aiReaction=0.8;
        }
    }

    // ---------- Paint ----------
    @Override
    protected void paintComponent(Graphics g){
        super.paintComponent(g);
        g.setColor(Color.BLACK);
        g.fillRect(0,0,WIDTH,HEIGHT);

        g.setColor(Color.WHITE);
        g.fillRect(10,p1Y,PADDLE_W,PADDLE_H);
        g.fillRect(WIDTH-25,p2Y,PADDLE_W,PADDLE_H);
        g.fillOval(ballX,ballY,BALL_SIZE,BALL_SIZE);

        g.setFont(new Font("Arial",Font.BOLD,20));
        g.drawString(p1Score + " - " + p2Score, WIDTH/2-20,30);
    }

    // ---------- Game loop ----------
    @Override
    public void actionPerformed(ActionEvent e){
        if(p1Score>=WIN_SCORE || p2Score>=WIN_SCORE) return;

        // Ball movement
        ballX += ballXV;
        ballY += ballYV;

        // Player movement
        int playerSpeed=6;
        if(upPressed && p2Y>0) p2Y-=playerSpeed;
        if(downPressed && p2Y<HEIGHT-PADDLE_H-30) p2Y+=playerSpeed;

        // ---------- AI ----------
        if(Math.random() < aiReaction){
            int targetY = ballY + BALL_SIZE/2 - PADDLE_H/2;
            int diff = targetY - p1Y;

            int step = Math.min(aiSpeed, Math.abs(diff));
            if(diff>0) p1Y += step;
            else if(diff<0) p1Y -= step;

            // small random error
            p1Y += (int)(Math.random()*2) - 1;
        }

        if(p1Y<0) p1Y=0;
        if(p1Y>HEIGHT-PADDLE_H-30) p1Y=HEIGHT-PADDLE_H-30;

        // Wall collision
        if(ballY<=0 || ballY>=HEIGHT-BALL_SIZE-30) ballYV=-ballYV;

        // Paddle collision
        if((ballX<=20 && ballY>=p1Y && ballY<=p1Y+PADDLE_H) ||
           (ballX>=WIDTH-35 && ballY>=p2Y && ballY<=p2Y+PADDLE_H)) ballXV=-ballXV;

        // Score
        if(ballX<0 || ballX>WIDTH){
            if(ballX<0) p2Score++; else p1Score++;
            ballX=WIDTH/2; ballY=HEIGHT/2;

            if(p1Score>=WIN_SCORE || p2Score>=WIN_SCORE) showPlayAgain();
        }

        repaint();
    }

    // ---------- Player controls ----------
    @Override
    public void keyPressed(KeyEvent e){
        if(e.getKeyCode()==KeyEvent.VK_W) upPressed=true;
        if(e.getKeyCode()==KeyEvent.VK_S) downPressed=true;
    }
    @Override
    public void keyReleased(KeyEvent e){
        if(e.getKeyCode()==KeyEvent.VK_W) upPressed=false;
        if(e.getKeyCode()==KeyEvent.VK_S) downPressed=false;
    }
    @Override public void keyTyped(KeyEvent e){}

    // ---------- Play Again ----------
    void showPlayAgain(){
        playAgainButton = new JButton("Play Again");
        playAgainButton.setBounds(WIDTH/2-75, HEIGHT/2-25,150,50);
        this.setLayout(null);
        this.add(playAgainButton);
        playAgainButton.addActionListener(e->resetGame());
        playAgainButton.setFocusable(false);
        this.repaint();
    }

    void resetGame(){
        p1Score=0; p2Score=0;
        ballX=WIDTH/2; ballY=HEIGHT/2;
        if(playAgainButton!=null){
            this.remove(playAgainButton);
            playAgainButton=null;
        }
        this.requestFocusInWindow();
        repaint();
    }

    // ---------- Main ----------
    public static void main(String[] args){
        launchLauncher();
    }

    // ---------- Launcher ----------
    static void launchLauncher(){
        JFrame menu = new JFrame("Pong Launcher");
        menu.setSize(600,500);
        menu.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        menu.setLayout(null);

        JLabel title = new JLabel("Pong Game Launcher", SwingConstants.CENTER);
        title.setBounds(0,20,600,50);
        title.setFont(new Font("Arial",Font.BOLD,24));
        menu.add(title);

        String[] difficulties={"Easy","Medium","Hard"};
        JComboBox<String> difficultyBox = new JComboBox<>(difficulties);
        difficultyBox.setBounds(220,90,150,25);
        menu.add(difficultyBox);

        JButton start = new JButton("Start Game");
        start.setBounds(220,130,150,40);
        menu.add(start);

        JButton exit = new JButton("Exit");
        exit.setBounds(220,180,150,40);
        menu.add(exit);

        JTextArea instructions = new JTextArea(
            "Controls:\n- Move right paddle: W/S keys\n- AI on left paddle\n\nObjective:\nScore 5 points before AI to win!"
        );
        instructions.setEditable(false);
        instructions.setBounds(150,240,300,100);
        instructions.setFont(new Font("Arial",Font.PLAIN,14));
        instructions.setBackground(menu.getBackground());
        menu.add(instructions);

        JLabel authorLabel = new JLabel("<html>Author: The Creator Of The Game<br>" +
                "<a href=''>YouTube: https://youtube.com/@Stick_factsyt</a></html>");
        authorLabel.setBounds(150,360,300,50);
        authorLabel.setCursor(new Cursor(Cursor.HAND_CURSOR));
        authorLabel.addMouseListener(new MouseAdapter(){
            @Override
            public void mouseClicked(MouseEvent e){
                try{ Desktop.getDesktop().browse(new URI("https://youtube.com/@Stick_factsyt")); }
                catch(Exception ex){ ex.printStackTrace(); }
            }
        });
        menu.add(authorLabel);

        start.addActionListener(e->{
            menu.dispose();
            launchAgeCheck((String)difficultyBox.getSelectedItem());
        });

        exit.addActionListener(e->System.exit(0));
        menu.setVisible(true);
    }

    // ---------- Age check ----------
    static void launchAgeCheck(String difficulty){
        JFrame ageFrame = new JFrame("Age Check");
        ageFrame.setSize(400,200);
        ageFrame.setLayout(null);
        ageFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JLabel ageLabel = new JLabel("Enter your age:");
        ageLabel.setBounds(30,30,120,25);
        ageFrame.add(ageLabel);

        JTextField ageField = new JTextField();
        ageField.setBounds(150,30,50,25);
        ageFrame.add(ageField);

        JButton submit = new JButton("Submit");
        submit.setBounds(210,30,100,25);
        ageFrame.add(submit);

        submit.addActionListener(e->{
            try{
                int age = Integer.parseInt(ageField.getText());
                if(age>=18){
                    int choice = JOptionPane.showConfirmDialog(ageFrame,"You are eligible! Continue to game?","Continue?",JOptionPane.YES_NO_OPTION);
                    if(choice==JOptionPane.YES_OPTION){
                        ageFrame.dispose();
                        launchPong(difficulty);
                    }
                } else{
                    JOptionPane.showMessageDialog(ageFrame,"You are not eligible to continue.","Blocked",JOptionPane.WARNING_MESSAGE);
                }
            }catch(Exception ex){
                JOptionPane.showMessageDialog(ageFrame,"Enter a valid number.","Error",JOptionPane.ERROR_MESSAGE);
            }
        });

        ageFrame.setVisible(true);
    }

    static void launchPong(String difficulty){
        JFrame pongFrame = new JFrame("Pong Game");
        PongGame game = new PongGame(pongFrame,difficulty);
        pongFrame.add(game);
        pongFrame.setSize(WIDTH,HEIGHT);
        pongFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        pongFrame.setVisible(true);
        game.requestFocusInWindow(); // ensures key controls work
    }
}