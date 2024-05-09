package Assignment3;

import java.io.BufferedWriter;
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import javax.imageio.ImageIO;
import org.openqa.selenium.By;
import org.openqa.selenium.JavascriptExecutor;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;

public class WebCrawler {
	
	//This function just operates to scroll to the bottom of the page slowly 
	public static void scrollToBottom(WebDriver driver) throws Exception {
        JavascriptExecutor js = (JavascriptExecutor) driver;
        
        int scrollLimit = 300;
        for (int i = 0; i <= scrollLimit; i += 5) {
            js.executeScript("window.scrollBy(0, " + i + ")", "");
            Thread.sleep(70);//thread.sleep allows slow scrolling
        }  
        driver.navigate().refresh();//refresh the page to go back up
        Thread.sleep(2000);
    }
	
	//This function scrapes the about page. It plays the youtube video that is embedded in the webpage
	private static void callAbout(WebDriver driver) throws Exception {
		WebElement iframe = driver.findElement(By.xpath("//*[@id='rootApp']/div[2]/div/div[3]/div/div/div[1]/iframe"));//find the button that plays the youtube video
		
		//Switch to that iframe and click the button
		driver.switchTo().frame(iframe);
		driver.findElement(By.xpath("//*[@id='movie_player']/div[4]/button")).click();
		Thread.sleep(9000);
		
		//switch back to the parent iframe
		driver.switchTo().parentFrame();
	}
	
	//This function iterates over every single petition that is visible and scrapes them
	private static void callPetitions(WebDriver driver) throws Exception {
		List<WebElement> linkElements = driver.findElements(By.className("link-block"));

        Map<String, String> linksMap = new HashMap<>();

        //Iterate and find all the elements with h4 tag and get their link
        for (WebElement linkElement : linkElements) {
            String linkText = linkElement.findElement(By.tagName("h4")).getText();
            String linkHref = linkElement.getAttribute("href");

            linksMap.put(linkText, linkHref);
        }
        
        //Iterate over every link obtained in the above loop and scrape them
        for (Map.Entry<String, String> entry : linksMap.entrySet()) {
        	navigateToWebpage(driver, entry.getValue().replace("https://www.change.org/", ""));
        	scapeTheArticle(driver, entry.getKey());
        	Thread.sleep(2000);
        	driver.navigate().back();
        }
	}
	
	//The article link obtained from the callPetitions function, are scraped for their text, link, and the image
	private static void scapeTheArticle(WebDriver driver, String linkText) throws Exception {
		
		FileWriter fileWriter = new FileWriter("/Users/ysk/eclipse-workspace/Selenium/src/ArticleText/" + linkText + ".txt");
        BufferedWriter writer = new BufferedWriter(fileWriter);
		WebElement divElement = driver.findElement(By.className("corgi-1wj70kl"));

        List<WebElement> paragraphElements = divElement.findElements(By.tagName("p"));
        
        //The text obtained is written into a txt file in the ArticleText directory
        for (WebElement paragraph : paragraphElements) {
            System.out.println(paragraph.getText());
            String paragraphText = paragraph.getText();
            writer.write(paragraphText);
            writer.newLine();
        }
        
        writer.close();
        
        WebElement imageElement = driver.findElement(By.cssSelector("img.corgi-upw6j2"));

        File screenshot = imageElement.getScreenshotAs(org.openqa.selenium.OutputType.FILE);

      //The image obtained is stored as a png file in the ArticleImage directory
        String destinationPath = "/Users/ysk/eclipse-workspace/Selenium/src/ArticlePhotos/" + linkText + ".png";
        try {
            ImageIO.write(ImageIO.read(screenshot), "png", new File(destinationPath));
            System.out.println("Image saved to: " + destinationPath);
        } catch (IOException e) {
            e.printStackTrace();
        }
	}
	
	//This function scrapes the impact webpage and clicks on the download report button that is available
	private static void callImpact(WebDriver driver) throws Exception {
		JavascriptExecutor js = (JavascriptExecutor) driver;  
		WebElement element = driver.findElement(By.xpath("//*[@id='rootApp']/div[2]/div/div[3]/div/a"));		
        js.executeScript("arguments[0].scrollIntoView();", element);// scroll until the download button is visible and click it
        element.click();
	}

	//This acts as a primary function that navigates to the webpages
	private static void navigateToWebpage(WebDriver driver, String pageLink) {
		driver.navigate().to("https://www.change.org/" + pageLink);
	}
	
	public static void main(String[] args) throws Exception {
		
		WebDriver driver = new ChromeDriver();
		
		//Open the home page
		navigateToWebpage(driver, "");
		
		//navigate to the about page
		navigateToWebpage(driver, "about");
		callAbout(driver);// This function scrapes the about page
		driver.navigate().back();//navigate back to the home page
		
		//navigate to the petitions page
		navigateToWebpage(driver, "petitions");
		callPetitions(driver);//iterate over all the petitions and scrape them
		navigateToWebpage(driver, "");//navigate back to the home page
		
		//navigate to the impact page
		navigateToWebpage(driver, "impact");
		scrollToBottom(driver);//scroll to the bottom
		callImpact(driver);//scroll to the mentioned button and click it and download it
		
		Thread.sleep(5000);
		driver.quit();//close the chrome instance
	}

}
