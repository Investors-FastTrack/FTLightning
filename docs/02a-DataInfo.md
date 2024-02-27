# Data Excellence at FastTrack
FastTrack prides itself on delivering precise, timely, and comprehensive financial data to empower your investment decisions. Our rigorous data collection process, frequent updates, and user-focused data formatting ensure you have the insights needed to excel in the dynamic financial landscape.

## Unparalleled Data Update Schedule

We understand the importance of having the latest information at your fingertips. That's why FastTrack updates our database three times daily, offering one of the industry's most frequent update schedules:

### Daily Update Cycle:
- **Publish 1** - 6:40 PM EST: The first update of the evening brings you the day's prices and dividends for a vast array of assets. This includes all stocks, ETFs, COFs, and a significant portion of open-end funds and indexes.

- **Publish 2** - 8:25 PM EST: Our second update finalizes the day's data, incorporating any late-reporting assets not covered in Publish 1.

- **Publish 3** - 9:00 PM EST: A specialized update focusing on ~100 bond index values, ensuring bond market participants have the latest data.

- **Publish 4** - 7:25 AM EST: Before the market opens, we provide overnight updates and dividends, ensuring you start the day with the most current data.

Stay informed on data updates through our [/v1/status](https://fasttrack.stoplight.io/docs/ftlightning/ad1cc0b0b84ef-data-update-status) endpoint.



### Ensuring Data Continuity:
- **Missing Prices:** We employ sophisticated methods to maintain continuity in our datasets. For any missing closing price, we default to the previous market day's value, ensuring no gaps in your analysis.

- **OLHV Data:** In the rare instance of missing open, high, low, or volume data, we mark these with "-1" to signify the absence, allowing for seamless data processing.

## Dividend Adjusted Prices for Accurate Analysis
At FastTrack, "price" means more than just a number. Our adjusted price data accounts for dividends, providing a true reflection of a security's value and ensuring your total return calculations are accurate. Unadjusted prices are also available, offering flexibility in analysis.

## Optimized Defaults for Efficient Data Access
Our API endpoints are designed with intelligent defaults, simplifying access while allowing for customized queries:

- **End Date**: Defaults to the latest data in our database, ensuring you're always working with current information.
- **Start Date**: Set to one year before the latest data, offering a comprehensive historical view.
- **Basis and Length**: Defaults such as SP-DA and 21 market days optimize for common analytical needs.
- **Count**: A default of 1000, balancing comprehensiveness and performance.
- **Risk-Free Rate**: The Vanguard money market fund (VMMXX) is our standard, reflecting a widely accepted benchmark.

---
## Why Choose FastTrack?
- **Timeliness**: With three daily updates, your decisions are powered by some of the freshest data available.
- **Reliability**: Our handling of missing data ensures consistency and reliability in your analyses.
- **Insight**: Dividend-adjusted prices provide deeper insights for more accurate investment strategies.
- **Ease of Use**: Intelligent defaults and comprehensive documentation make our API user-friendly and accessible.

FastTrack is committed to delivering not just data, but a competitive edge. Explore our offerings and see why leading professionals trust FastTrack for their financial data needs.




