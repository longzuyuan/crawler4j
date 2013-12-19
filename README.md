crawler4j
=========

Open Source Web Crawler for Java. 
Clone from https://code.google.com/p/crawler4j/

Crawler4j is an open source Java crawler which provides a simple interface for crawling the Web. You can setup a multi-threaded web crawler in 5 minutes!

Sample Usage
You need to create a crawler class that extends WebCrawler. This class decides which URLs should be crawled and handles the downloaded page. The following is a sample implementation:

public class MyCrawler extends WebCrawler {

        private final static Pattern FILTERS = Pattern.compile(".*(\\.(css|js|bmp|gif|jpe?g" 
                                                          + "|png|tiff?|mid|mp2|mp3|mp4"
                                                          + "|wav|avi|mov|mpeg|ram|m4v|pdf" 
                                                          + "|rm|smil|wmv|swf|wma|zip|rar|gz))$");

        /**
         * You should implement this function to specify whether
         * the given url should be crawled or not (based on your
         * crawling logic).
         */
        @Override
        public boolean shouldVisit(WebURL url) {
                String href = url.getURL().toLowerCase();
                return !FILTERS.matcher(href).matches() && href.startsWith("http://www.ics.uci.edu/");
        }

        /**
         * This function is called when a page is fetched and ready 
         * to be processed by your program.
         */
        @Override
        public void visit(Page page) {          
                String url = page.getWebURL().getURL();
                System.out.println("URL: " + url);

                if (page.getParseData() instanceof HtmlParseData) {
                        HtmlParseData htmlParseData = (HtmlParseData) page.getParseData();
                        String text = htmlParseData.getText();
                        String html = htmlParseData.getHtml();
                        List<WebURL> links = htmlParseData.getOutgoingUrls();

                        System.out.println("Text length: " + text.length());
                        System.out.println("Html length: " + html.length());
                        System.out.println("Number of outgoing links: " + links.size());
                }
        }
}
As can be seen in the above code, there are two main functions that should be overridden:

shouldVisit: This function decides whether the given URL should be crawled or not. In the above example, this example is not allowing .css, .js and media files and only allows pages within 'www.ics.uci.edu' domain.
visit: This function is called after the content of a URL is downloaded successfully. You can easily get the url, text, links, html, and unique id of the downloaded page.
You should also implement a controller class which specifies the seeds of the crawl, the folder in which intermediate crawl data should be stored and number of concurrent threads:

public class Controller {
        public static void main(String[] args) throws Exception {
                String crawlStorageFolder = "/data/crawl/root";
                int numberOfCrawlers = 7;

                CrawlConfig config = new CrawlConfig();
                config.setCrawlStorageFolder(crawlStorageFolder);

                /*
                 * Instantiate the controller for this crawl.
                 */
                PageFetcher pageFetcher = new PageFetcher(config);
                RobotstxtConfig robotstxtConfig = new RobotstxtConfig();
                RobotstxtServer robotstxtServer = new RobotstxtServer(robotstxtConfig, pageFetcher);
                CrawlController controller = new CrawlController(config, pageFetcher, robotstxtServer);

                /*
                 * For each crawl, you need to add some seed urls. These are the first
                 * URLs that are fetched and then the crawler starts following links
                 * which are found in these pages
                 */
                controller.addSeed("http://www.ics.uci.edu/~welling/");
                controller.addSeed("http://www.ics.uci.edu/~lopes/");
                controller.addSeed("http://www.ics.uci.edu/");

                /*
                 * Start the crawl. This is a blocking operation, meaning that your code
                 * will reach the line after this only when crawling is finished.
                 */
                controller.start(MyCrawler.class, numberOfCrawlers);    
        }
}
Example Codes
Basic crawler: the full source code of the above example with more details.
Image crawler: a simple image crawler that downloads image content from the crawling domain and stores them in a folder. This example demonstrates how binary content can be fetched using crawler4j.
Collecting data from threads: this example demonstrates how the controller can collect data/statistics from crawling threads.
Multiple crawlers: this is a sample that shows how two distinct crawlers can run concurrently. For example, you might want to split your crawling into different domains and then take different crawling policies for each group. Each crawling controller can have its own configurations.
Shutdown crawling: this example shows have crawling can be terminated gracefully by sending the 'shutdown' command to the controller.
More details
Configuration details
Maven dependencies
Frequently asked questions
