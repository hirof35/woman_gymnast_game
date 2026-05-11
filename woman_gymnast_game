package ImageGymnasticsGame;

import java.awt.Color;
import java.awt.Dimension;
import java.awt.Font;
import java.awt.GradientPaint;
import java.awt.Graphics;
import java.awt.Graphics2D;
import java.awt.RenderingHints;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;
import java.awt.geom.AffineTransform;
import java.awt.image.BufferedImage;
import java.io.File;
import java.util.ArrayList;
import java.util.List;

import javax.imageio.ImageIO;
import javax.swing.JFrame;
import javax.swing.JPanel;

public class ImageGymnasticsGame extends JPanel implements Runnable {
    // --- 定数設定 ---
    private static final int WIDTH = 1000;
    private static final int HEIGHT = 600;
    private static final int GROUND_Y = 500;
    private static final String IMG_PREFIX = "image_";

    // --- ゲーム状態 ---
    private enum State { MENU, PLAYING, RESULT }
    private State currentState = State.MENU;

    // --- プレイヤーデータ構造 ---
    private static class PlayerData {
        String name;
        BufferedImage standImg, runImg1, runImg2, jumpImg;
        double speedMult, jumpPower, rotMult;

        public PlayerData(String name, int id, double s, double j, double r) {
            this.name = name;
            this.speedMult = s;
            this.jumpPower = j;
            this.rotMult = r;
            
            // 各種アクション画像の読み込み
            this.standImg = loadImage(IMG_PREFIX + id + ".png");
            this.runImg1 = loadImage(IMG_PREFIX + id + "_run1.png");
            this.runImg2 = loadImage(IMG_PREFIX + id + "_run2.png");
            this.jumpImg = loadImage(IMG_PREFIX + id + "_jump.png");

            // 画像が1枚もない場合の予備処理（静止画へ流用）
            if (runImg1 == null) runImg1 = standImg;
            if (runImg2 == null) runImg2 = standImg;
            if (jumpImg == null) jumpImg = standImg;
        }

        private BufferedImage loadImage(String path) {
            try {
                File f = new File(path);
                if (!f.exists()) f = new File("src/" + path);
                return f.exists() ? ImageIO.read(f) : null;
            } catch (Exception e) { return null; }
        }
    }

    private List<PlayerData> playerList = new ArrayList<>();
    private int selectedPlayerIndex = 0;
    private PlayerData currentPlayer;
    
    // --- アニメーション・物理変数 ---
    private int animFrame = 0;
    private int animTimer = 0;
    private double x = 50, y = GROUND_Y, speed = 0, velocityY = 0;
    private double angle = 0, twistAngle = 0, rotSpeed = 0, twistSpeed = 0;
    private boolean isJumping = false, lastKeyLeft = false;
    private String resultText = "";
    private double score = 0;

    public ImageGymnasticsGame() {
        setPreferredSize(new Dimension(WIDTH, HEIGHT));
        setFocusable(true);
        setBackground(new Color(10, 10, 25));

        // 6人の選手を初期化（名前と能力値：速度、跳躍力、回転率）
        String[] names = {"Miyakawa", "Murakami", "Hatakeda", "Sugihara", "Ashikawa", "Hiraiwa"};
        double[][] stats = {
            {1.1, 0.82, 0.06}, {1.2, 0.85, 0.07}, {1.0, 0.92, 0.05},
            {1.3, 0.78, 0.08}, {1.1, 0.80, 0.06}, {1.2, 0.88, 0.07}
        };

        for (int i = 0; i < 6; i++) {
            playerList.add(new PlayerData(names[i], i, stats[i][0], stats[i][1], stats[i][2]));
        }
        currentPlayer = playerList.get(0);

        addKeyListener(new KeyAdapter() {
            @Override public void keyPressed(KeyEvent e) { handleInput(e.getKeyCode()); }
        });
        new Thread(this).start();
    }

    private void handleInput(int code) {
        if (currentState == State.MENU) {
            if (code == KeyEvent.VK_LEFT) selectedPlayerIndex = (selectedPlayerIndex - 1 + 6) % 6;
            if (code == KeyEvent.VK_RIGHT) selectedPlayerIndex = (selectedPlayerIndex + 1) % 6;
            currentPlayer = playerList.get(selectedPlayerIndex);
            if (code == KeyEvent.VK_ENTER) { resetGame(); currentState = State.PLAYING; }
        } else if (currentState == State.PLAYING && !isJumping) {
            // 交互連打システム
            if ((code == KeyEvent.VK_LEFT && !lastKeyLeft) || (code == KeyEvent.VK_RIGHT && lastKeyLeft)) {
                speed += 1.6 * currentPlayer.speedMult;
                lastKeyLeft = !lastKeyLeft;
            }
            // 踏切（スペースキー）
            if (code == KeyEvent.VK_SPACE && x > 400) {
                isJumping = true;
                velocityY = -speed * currentPlayer.jumpPower;
                rotSpeed = speed * currentPlayer.rotMult;
                twistSpeed = 0.28;
            }
        } else if (currentState == State.RESULT && code == KeyEvent.VK_ENTER) {
            currentState = State.MENU;
        }
    }

    private void resetGame() {
        x = 50; y = GROUND_Y; speed = 0; velocityY = 0;
        angle = 0; twistAngle = 0; isJumping = false;
        animTimer = 0; animFrame = 0;
    }

    private void update() {
        if (currentState != State.PLAYING) return;

        if (!isJumping) {
            speed *= 0.97; // 摩擦
            x += speed;
            if (x > 550) x = 550;

            // 走行アニメーション制御
            if (speed > 0.1) {
                animTimer += (int)speed;
                if (animTimer > 12) {
                    animFrame = (animFrame == 0) ? 1 : 0;
                    animTimer = 0;
                }
            }
        } else {
            // 空中物理計算
            x += speed * 0.35;
            y += velocityY;
            velocityY += 0.58; // 重力
            angle += rotSpeed;
            twistAngle += twistSpeed;

            if (y >= GROUND_Y) {
                y = GROUND_Y;
                isJumping = false;
                evaluateLanding();
                currentState = State.RESULT;
            }
        }
    }

    private void evaluateLanding() {
        double d = Math.abs(angle % (Math.PI * 2));
        double err = Math.min(d, Math.PI * 2 - d);
        if (err < 0.4) { resultText = "PERFECT STICK!"; score = 10.0; }
        else if (err < 1.1) { resultText = "GOOD LANDING"; score = 8.5; }
        else { resultText = "CRASH / FALL"; score = 5.0; }
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2d = (Graphics2D) g;
        g2d.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);

        // 背景描画
        g2d.setPaint(new GradientPaint(0, 0, new Color(15, 15, 40), 0, HEIGHT, new Color(40, 40, 80)));
        g2d.fillRect(0, 0, WIDTH, HEIGHT);

        if (currentState == State.MENU) drawMenu(g2d);
        else drawGame(g2d);
    }

    private void drawMenu(Graphics2D g2d) {
        g2d.setColor(Color.WHITE);
        g2d.setFont(new Font("Arial", Font.BOLD, 40));
        g2d.drawString("SELECT GYMNAST", 320, 80);

        for (int i = 0; i < 6; i++) {
            int px = 45 + i * 158;
            int py = 180;
            if (i == selectedPlayerIndex) {
                g2d.setColor(new Color(255, 255, 100, 80));
                g2d.fillRoundRect(px - 5, py - 5, 145, 280, 20, 20);
                g2d.setColor(Color.YELLOW);
                g2d.drawRoundRect(px - 5, py - 5, 145, 280, 20, 20);
            }
            
            PlayerData p = playerList.get(i);
            if (p.standImg != null) {
                g2d.drawImage(p.standImg, px, py, 135, 270, null);
            } else {
                g2d.setColor(Color.DARK_GRAY);
                g2d.fillRect(px, py, 135, 270);
                g2d.setColor(Color.WHITE);
                g2d.drawString("P" + (i+1), px + 50, py + 140);
            }
        }
        g2d.setColor(Color.WHITE);
        g2d.setFont(new Font("Arial", Font.ITALIC, 20));
        g2d.drawString("Selected: " + currentPlayer.name, 400, 520);
        g2d.drawString("Press ENTER to Start Vault!", 370, 550);
    }

    private void drawGame(Graphics2D g2d) {
        // 床と踏切板
        g2d.setColor(new Color(60, 120, 220));
        g2d.fillRect(0, GROUND_Y, WIDTH, 100);
        g2d.setColor(Color.ORANGE);
        g2d.fillRect(520, GROUND_Y - 5, 70, 10);

        // 選手の描画
        AffineTransform old = g2d.getTransform();
        g2d.translate(x, y);
        g2d.rotate(angle);
        g2d.scale(Math.cos(twistAngle), 1.0);
        
        BufferedImage img;
        if (isJumping) img = currentPlayer.jumpImg;
        else if (speed > 0.5) img = (animFrame == 0) ? currentPlayer.runImg1 : currentPlayer.runImg2;
        else img = currentPlayer.standImg;

        if (img != null) {
            g2d.drawImage(img, -40, -180, 80, 180, null);
        } else {
            g2d.setColor(Color.RED);
            g2d.fillRect(-25, -150, 50, 150);
        }
        g2d.setTransform(old);

        // UI
        g2d.setColor(Color.WHITE);
        g2d.setFont(new Font("Monospaced", Font.BOLD, 22));
        g2d.drawString("SPEED: " + String.format("%.1f", speed), 30, 40);
        
        if (currentState == State.RESULT) {
            g2d.setColor(new Color(0,0,0,200));
            g2d.fillRoundRect(300, 200, 400, 160, 20, 20);
            g2d.setColor(Color.YELLOW);
            g2d.setFont(new Font("Arial", Font.BOLD, 35));
            g2d.drawString(resultText, 350, 260);
            g2d.setFont(new Font("Arial", Font.PLAIN, 25));
            g2d.drawString("Score: " + score, 430, 310);
        }
    }

    public void run() {
        while (true) {
            update();
            repaint();
            try { Thread.sleep(16); } catch (Exception e) {}
        }
    }

    public static void main(String[] args) {
        JFrame f = new JFrame("Team Gymnastics: Vault Championship");
        f.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        f.add(new ImageGymnasticsGame());
        f.pack();
        f.setLocationRelativeTo(null);
        f.setVisible(true);
    }
}
