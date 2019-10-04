<p>目录</p>
<a href='https://github.com/DavidAwing/davidawing.github.io/blob/master/AlarmClock.md#alarmclockmainjava'>AlarmClockMain</a></br>
<a href='https://github.com/DavidAwing/davidawing.github.io/blob/master/AlarmClock.md#timetoanglejava'>AlarmClock</a>
</br>
<a href='https://github.com/DavidAwing/davidawing.github.io/blob/master/AlarmClock.md#playsoundthreadjava'>PlaySoundThread</a>
</br>
<a href='https://github.com/DavidAwing/davidawing.github.io/blob/master/AlarmClock.md#drawthreadjava'>DrawThread</a>
</br>
<a href='https://github.com/DavidAwing/davidawing.github.io/blob/master/AlarmClock.md#alamclockjava'>AlamClock</a>


<img scr='https://raw.githubusercontent.com/DavidAwing/davidawing.github.io/master/image/AlarmClock.PNG'/>


<h1>AlarmClockMain.java</h1>

```java

package com.wing.AlarmClock;

import java.awt.*;

public class AlarmClockMain {

	public static void main(String[] args) {

		int width = 800, height = 800;
		AlamClock alarmClock = new AlamClock(width/2, height/2);
		alarmClock.setSize(width, height);
		alarmClock.setVisible(true);
		alarmClock.setResizable(false);
		Dimension screen = Toolkit.getDefaultToolkit().getScreenSize();
		//设置窗口的位置到屏幕中心
		alarmClock.setLocation((screen.width-width)/2,(screen.height-height)/2);
		alarmClock.start();
		System.out.println("finish");
	}
}


```


<h1>TimeToAngle.java</h1>

```java
 
package com.wing.AlarmClock;

import java.util.Calendar;
import java.util.GregorianCalendar;

//时间转化为角度
public class TimeToAngle {

	public void main(String[] args) {
		System.out.println(getCurrentHour());
		System.out.println(getCurrentMinute());
		System.out.println(getCurrentSecond());
	}
	
	//获取当前小时
	public int getCurrentHour() {
		return new GregorianCalendar().get(Calendar.HOUR_OF_DAY);
	}
	
	//获取当前分钟
	public int getCurrentMinute() {
		return new GregorianCalendar().get(Calendar.MINUTE);
	}
	
	//获取当前秒
	public int getCurrentSecond() {
		return new GregorianCalendar().get(Calendar.SECOND);
	}
	
	//获取当前时针的角度
	public int currentHourToAngle() {
		int angle = -1;
		angle = (int)(getCurrentHour() * 30 + getCurrentMinute() * 0.5 - 90);
		return angle;
	}
	
	//获取当前分针的角度
	public int currentMinuteToAngle() {
		int angle = -1;
		angle = getCurrentMinute() * 6 - 90;
		return angle;
	}
	
	//获取当前秒针的角度
	public int currentSecondToAngle() {
		int angle = -1;
		angle = getCurrentSecond() * 6 - 90;
		return angle;
	}

} 
```

<h1>PlaySoundThread.java</h1>


```java

package com.wing.AlarmClock;
//播放声音

import java.io.BufferedInputStream;
import java.io.InputStream;

import javazoom.jl.player.Player;

public class PlaySoundThread extends Thread {
	String filename;

	public PlaySoundThread(String filename) {
		this.filename = filename;
	}

	@Override
	public void run() {
		while (true) {
			play(filename);
		}
	}

	public void play(String filename) {
		try {
			InputStream input = this.getClass().getResourceAsStream("/sound/dida.mp3");  
			BufferedInputStream buffer = new BufferedInputStream(input);
			Player player = new Player(buffer);
			player.play();
		} catch (Exception e) {
			System.out.println(e);
		}
	}

}

```

<h1>DrawThread.java</h1>

```java
package com.wing.AlarmClock;

import javax.swing.*;

//驱动重绘视图
public class DrawThread extends Thread {

	JFrame jFrame;
	int millis;

	public DrawThread(JFrame jFrame) {
		this.jFrame = jFrame;
		this.millis = 1000;
	}

	public DrawThread(JFrame jFrame, int millis) {
		this.jFrame = jFrame;
		this.millis = millis;
	}

	@Override
	public void run() {
		while (true) {
			try {
				jFrame.update(jFrame.getGraphics());
				Thread.sleep(millis);
			} catch (InterruptedException e) {
				e.printStackTrace();
			} 
		}
	}

}
```


<h1>AlamClock.java</h1>

```java
package com.wing.AlarmClock;

import java.awt.*;
import java.awt.event.WindowEvent;
import java.awt.geom.Rectangle2D;

//绘制闹钟
public class AlamClock extends javax.swing.JFrame {

    // swing绘制： repaint => update => paint

    private float timeYPosition = 0.2f;//时间字符显示Y轴位置,建议取值1到-1之间

    private int cx, cy;//闹钟的中心
    private int rotateAngle = 360;

    // 双缓冲
    private Image iBuffer;
    private Graphics gBuffer;

    public AlamClock(int cx, int cy) {
        super("时钟");
        this.cx = cx;
        this.cy = cy;
    }

    public void start() {

        //驱动界面重绘的线程
        new DrawThread(this).start();

        // 播放声音的线程
        new PlaySoundThread("dida.mp3").start();
    }

    @Override
    protected void processWindowEvent(WindowEvent e) {
        System.out.println("processWindowEvent");
        if (e.getID() == WindowEvent.WINDOW_CLOSING) {
            System.exit(0);
        }
    }

    @Override
    public void update(Graphics g) {

        if (iBuffer == null) {
            //创建可双缓冲的Image对象
            iBuffer = createImage(this.getSize().width, this.getSize().height);
            gBuffer = iBuffer.getGraphics();
        }

        gBuffer.setColor(getBackground());
        gBuffer.fillRect(0, 0, this.getSize().width, this.getSize().height);

        paint(gBuffer);
        g.drawImage(iBuffer, 0, 0, this);
    }

    @Override
    public void paint(Graphics g) {

        int insideRadius = (int) (this.getWidth() / 2 * 0.5);//闹钟框内圆
        int outside = (int) (this.getWidth() / 2 * 0.58);//闹钟框外圆

        TimeToAngle timeToAngle = new TimeToAngle();

        drawCircle(g, Color.BLACK, insideRadius);
        drawCircle(g, Color.BLACK, outside);
        drawHourScale(g, Color.RED, insideRadius + 3, outside - 3);
        drawMinuteScale(g, Color.BLUE, insideRadius + 6, outside - 6);

        drawHourHand(g, Color.BLACK, insideRadius - 36, timeToAngle.currentHourToAngle(), -10, 25, 20);
        drawHourTxt(g, Color.BLACK, insideRadius - 10);

        drawHourHand(g, Color.BLACK, insideRadius - 16, timeToAngle.currentMinuteToAngle(), -20, 25, 20);

        drawHourHand(g, Color.RED, insideRadius - 3, timeToAngle.currentSecondToAngle(), -30, 25, 20);


        drawEar(g, outside);

        drawBracket(g, Color.BLACK, outside);


        int h = timeToAngle.getCurrentHour();
        int m = timeToAngle.getCurrentMinute();
        int s = timeToAngle.getCurrentSecond();
        drawTime(g, Color.blue, cx, cy - (int) (outside * timeYPosition),
                (h < 10 ? "0" + h : h) + ":" + (m < 10 ? "0" + m : m) + ":" + (s < 10 ? "0" + s : s));
    }

    /**
     * 画一个圆
     *
     * @param color 颜色
     * @param r     圆半径
     */
    private void drawCircle(Graphics g, Color color, int r) {
        g.setColor(color);

        for (int i = 0; i < rotateAngle; i++) {
            int x = (int) (r * Math.cos(i * Math.PI / 180) + cx);
            int y = (int) (r * Math.sin(i * Math.PI / 180) + cy);
            g.drawLine(x, y, x, y);
        }
    }

    /**
     * 画小时刻度
     *
     * @param color        颜色
     * @param insideRadius 内圆半径
     * @param outside      外圆半径
     */
    private void drawHourScale(Graphics g, Color color, int insideRadius, int outside) {
        g.setColor(color);
        for (int i = insideRadius; i < outside; i++) {
            for (int j = 0; j < rotateAngle; j += 30) {
                int x = (int) (i * Math.cos(j * Math.PI / 180) + cx);
                int y = (int) (i * Math.sin(j * Math.PI / 180) + cy);
                g.drawLine(x, y, x, y);

            }
        }
    }

    /**
     * @param g
     * @param color
     * @param insideRadius 字体环绕圆的半径
     */
    private void drawHourTxt(Graphics g, Color color, int insideRadius) {
        g.setColor(color);

        int x, y;

        for (int i = 0; i < 12; i++) {
            x = (int) (insideRadius * Math.cos(30 * (i - 2) * Math.PI / 180) + cx)-5;
            y = (int) (insideRadius * Math.sin(30 * (i - 2) * Math.PI / 180) + cy)+5;

            g.drawString(i + 1 + "", x, y);
        }
    }

    /**
     * 画分钟刻度
     *
     * @param color        颜色
     * @param insideRadius 内圆半径
     * @param outside      外圆半径
     */
    private void drawMinuteScale(Graphics g, Color color, int insideRadius, int outside) {
        g.setColor(color);
        for (int i = insideRadius; i < outside; i++) {
            for (int j = 0; j < rotateAngle; j += 6) {

                if (j % 30 == 0)
                    continue;

                int x = (int) (i * Math.cos(j * Math.PI / 180) + cx);
                int y = (int) (i * Math.sin(j * Math.PI / 180) + cy);
                g.drawLine(x, y, x, y);
            }
        }
    }

    /**
     * 画指针
     *
     * @param color       颜色
     * @param r           指针长度（半径）
     * @param handAngle   指针角度
     * @param tailLength  指针尾端长度
     * @param arrowAngle  箭头角度
     * @param arrowLength 箭头长度
     */
    private void drawHourHand(Graphics g, Color color, int r, int handAngle, int tailLength, int arrowAngle,
                              int arrowLength) {

        g.setColor(color);
        for (int i = tailLength; i <= r; i++) {

            // 画指针
            int x0 = (int) (i * Math.cos(handAngle * Math.PI / 180) + cx);
            int y0 = (int) (i * Math.sin(handAngle * Math.PI / 180) + cy);
            g.drawLine(x0, y0, x0, y0);

            // 画箭头
            if (i == r) {
                for (int k = 0; k < arrowLength; k++) {
                    int x = (int) (k * Math.cos((handAngle + (180 - arrowAngle)) * Math.PI / 180) + x0);
                    int y = (int) (k * Math.sin((handAngle + (180 - arrowAngle)) * Math.PI / 180) + y0);
                    g.drawLine(x, y, x, y);

                    x = (int) (k * Math.cos((handAngle - (180 - arrowAngle)) * Math.PI / 180) + x0);
                    y = (int) (k * Math.sin((handAngle - (180 - arrowAngle)) * Math.PI / 180) + y0);
                    g.drawLine(x, y, x, y);
                }
            }
        }
    }

    public void drawTime(Graphics g, Color color, int x, int y, String time) {
        Font preFont = g.getFont();

        Font font = new Font("黑体", Font.BOLD, 26);
        g.setFont(font);

        FontMetrics fm = g.getFontMetrics(font);
        Rectangle2D r2 = fm.getStringBounds(time, g);

        g.setColor(color);
        g.drawString(time, x - (int) r2.getCenterX(), y - (int) r2.getHeight() / 2);

        //画时间字符串外框
        g.drawRect(x - (int) r2.getWidth() / 2 - 10, y - (int) r2.getHeight() - 15, (int) r2.getWidth() + 20, (int) r2.getHeight() + 10);

        g.setFont(preFont);
    }


    //画支脚
    public void drawBracket(Graphics g, Color color, int r) {

        int rightAngle = 70;
        int leftAngle = 180 - rightAngle;
        int length = 80;

        g.setColor(color);

        for (int i = r; i <= r + length; i++) {
            int x = (int) (i * Math.cos(rightAngle * Math.PI / 180) + cx);
            int y = (int) (i * Math.sin(rightAngle * Math.PI / 180) + cy);
            g.drawLine(x, y, x, y);
        }

        {
            int lineLength = 100;
            int lx = (int) ((r + length) * Math.cos(leftAngle * Math.PI / 180) + cx) - lineLength / 2;
            int rx = (int) ((r + length) * Math.cos(rightAngle * Math.PI / 180) + cx) - lineLength / 2;
            int y = (int) ((r + length) * Math.sin(rightAngle * Math.PI / 180) + cy);

            for (; lineLength > 0; lineLength--) {
                g.drawLine(rx + lineLength, y, rx + lineLength, y);
                g.drawLine(lx + lineLength, y, lx + lineLength, y);
            }
        }

        for (int i = r; i <= r + length; i++) {
            int x = (int) (i * Math.cos(leftAngle * Math.PI / 180) + cx);
            int y = (int) (i * Math.sin(leftAngle * Math.PI / 180) + cy);
            g.drawLine(x, y, x, y);
        }
    }


    //画耳朵
    public void drawEar(Graphics g, int r) {
        Font formerFont = g.getFont();

        g.setFont(new Font("宋体",Font.PLAIN,100));
        int radii = 50;//半径
        int a = -65;

        {
            int x = (int) ((r + radii) * Math.cos(a * Math.PI / 180) + cx);
            int y = (int) ((r + radii) * Math.sin(a * Math.PI / 180) + cy);
            g.drawString("*",x-radii/2, y+radii/2);
            g.drawOval(x-radii, y-radii, radii*2, radii*2);
        }

        {
            int x = (int) ((r + radii) * Math.cos((-180-a) * Math.PI / 180) + cx);
            int y = (int) ((r + radii) * Math.sin((-180-a) * Math.PI / 180) + cy);
            g.drawString("*",x-radii/2, y+radii/2);
            g.drawOval(x-radii, y-radii, radii*2, radii*2);
        }

        g.setFont(formerFont);
    }
}

```



