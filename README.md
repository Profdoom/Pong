# Pong
Pong program for Java Programming Class.
Changed to add some features.
package com.EdwinDumalag;

import javax.swing.*;
import java.awt.*;
import java.awt.Color;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.KeyEvent;
import java.awt.event.KeyListener;
import java.io.File;
import java.io.IOException;


//TODO have paddle speed affect ball's direction
//TODO known issue - sometimes ball gets stuck behind human paddle

public class Pong {

    static gameDisplay.GameDisplay gamePanel;   //draw the game components here

    static boolean gameOver;      //Used to work out what message, if any, to display on the screen
    static boolean removeInstructions = false;  // Same as above
    boolean hitWall = false;
    boolean hitHumanPaddle = false;
    boolean hitComputerPaddle = false;

    static int screenSize = 300;    //and width - screen is square
    static int paddleSize = 25;     //Actually half the paddle size - how much to draw on each side of center
    static int paddleDistanceFromSide = 10;  //How much space between each paddle and side of screen

    static int gameSpeed = 75;  //How many milliseconds between clock ticks? Reduce this to speed up game

    static int computerPaddleY = screenSize / 2;    //location of the center of the paddles on the Y-axis of the screen
    static int humanPaddleY = screenSize / 2;

    static int computerPaddleMaxSpeed = 3;   //Max number of pixels that computer paddle can move clock tick. Higher number = easier for computer
    static int humanPaddleMaxSpeed = 5;   //This doesn't quite do the same thing... this is how many pixels human moves per key press TODO use this in a better way

    static int humanPaddleSpeed = 0;      // "speed" is pixels moved up or down per clock tick
    static int computerPaddleSpeed = 0;   // same

    static double ballX = screenSize / 2;//Imagine the ball is in a square box. These are the coordinates of the top of that box.

    static double ballY = screenSize / 2;   //So this starts the ball in (roughly) the center of the screen
    static int ballSize = 10;                //Diameter of ball

    static double ballSpeed = 5;   //Again, pixels moved per clock tick


    //An angle in radians (which range from 0 to 2xPI (0 to about 6.3).
    //This starts the ball moving down toward the human. Replace with some of the other
    //commented out versions for a different start angle
    //set this to whatever you want (but helps if you angle towards a player)
    static double ballDirection = Math.PI + 1;   //heading left and up
    //static double ballDirection = 1;
    //static double ballDirection = 0;   //heading right
    //static double ballDirection = Math.PI;   //heading left

    static Timer timer;    //Ticks every *gameSpeed* milliseconds. Every time it ticks, the ball and computer paddle move


    //Listen for user pressing a key, and moving human paddle in response
    private static class KeyHandler implements KeyListener {

        @Override
        public void keyTyped(KeyEvent ev) {
            char keyPressed = ev.getKeyChar();
            char q = 'q';
            if (keyPressed == q) {
                System.exit(0);    //quit if user presses the q key.
            }
        }

        @Override
        public void keyReleased(KeyEvent ev) {
        }   //Don't need this one, but required to implement it.

        @Override
        public void keyPressed(KeyEvent ev) {

            removeInstructions = true;   //game has started

            if (ev.getKeyCode() == KeyEvent.VK_DOWN) {
                System.out.println("down key");
                moveDown();
            }
            if (ev.getKeyCode() == KeyEvent.VK_UP) {
                System.out.println("up key");
                moveUp();
            }

            //ev.getComponent() returns the GUI component that generated this event
            //In this case, it will be GameDisplay JPanel
            ev.getComponent().repaint();   //This calls paintComponent(Graphics g) again
        }

        private void moveDown() {
            //Coordinates decrease as you go up the screen, that's why this looks backwards.
            if (humanPaddleY < screenSize - paddleSize) {
                humanPaddleY += humanPaddleMaxSpeed;
            }
        }

        private void moveUp() {
            //Coordinates increase as you go down the screen, that's why this looks backwards.
            if (humanPaddleY > paddleSize) {
                humanPaddleY -= humanPaddleMaxSpeed;
            }
        }

    }


    public static void main(String[] args) {

        gamePanel = new gameDisplay.GameDisplay();

        JPanel content = new JPanel();
        content.setLayout(new BorderLayout());
        content.add(gamePanel, BorderLayout.CENTER);

        JFrame window = new JFrame();
        window.setUndecorated(true);   //Hides the title bar.

        window.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);   //Quit the program when we close this window
        window.setContentPane(content);
        window.setSize(screenSize, screenSize);
        window.setLocation(100, 100);    //Where on the screen will this window appear?
        window.setVisible(true);

        KeyHandler listener = new KeyHandler();
        window.addKeyListener(listener);

        //Below, we'll create and start a timer that notifies an ActionListener every time it ticks
        //First, need to create the listener:
        ActionListener gameUpdater = new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {

                //gameUpdater is an inner class
                //It's containing class is Main
                //moveBall() and moveComputerPaddle belong to the outer class - Main
                //So we have to say Main.moveBall() to refer to these methods
                ballMove.moveBall();
                Pong.moveComputerPaddle();

                if (gameOver) {
                    timer.stop();
                }
                gamePanel.repaint();
            }
        };

        timer = new Timer(gameSpeed, gameUpdater);
        timer.start();    //Every time the timer ticks, the actionPerformed method of the ActionListener is called
    }

    //Uses the current position of ball and paddle to move the computer paddle towards the ball
    protected static void moveComputerPaddle() {

        //if ballY = 100 and paddleY is 50, difference = 50, need to adjust
        //paddleY by up to the max speed (the minimum of difference and maxSpeed)

        //if ballY = 50 and paddleY = 100 then difference = -50
        //Need to move paddleY down by the max speed

        int ballPaddleDifference = computerPaddleY - (int) ballY;
        int distanceToMove = Math.min(Math.abs(ballPaddleDifference), computerPaddleMaxSpeed);

        System.out.println("computer paddle speed = " + computerPaddleSpeed);

        if (ballPaddleDifference > 0) {   //Difference is positive - paddle is below ball on screen
            computerPaddleY -= distanceToMove;

        } else if (ballPaddleDifference < 0) {
            computerPaddleY += distanceToMove;

        } else {
            //Ball and paddle are aligned. Don't need to move!
            computerPaddleSpeed = 0;
        }

    }
}

    //Checks to see if the ball has hit a wall or paddle
    //If so, bounce off the wall/paddle
    //And then move ball in the correct direction
