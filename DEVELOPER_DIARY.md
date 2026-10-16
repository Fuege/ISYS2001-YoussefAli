# Developer's diary

I am starting this diary today on the 28/09/2026 to start tracking my progress for this assignment and project.

## 28 September 2026 — Choose the problem and design the app

Choose the Problem: The problem i have chosen is that small purchases are hard to track alongside large bills. So a university student would need a student spending and saving coach to assist them. The student needs a clear spending summary and a realistic weekly savings amount, rather than a generic answer that ignores their transactions.

**Objective:** build a student spending and savings coach using the techniques in the unit.

**Decisions recorded before implementation:** im recording my decisions before implementation before hand to keep the problem narrow; i need to represent money as integer cents; describe the upload as cash flow rather than assume it is a complete monthly budget; use a custom savings calculation; keep API keys in Colab Secrets; and use requests for the Gemini call as required by the brief.

**What was checked:** im looking back at my existing Module 07 and 08 work, and the Module 09, 10 and 11 course examples provided in the unit repo given to us to read for how to use Gradio, error handling and assertion patterns

**Next:**  Identify inputs and outputs to help later on.

## 29 September 2026 — Choosing my inputs and outputs

I worked on defining the inputs and outputs for the finance assistant before starting development. This will help me work out what information the program should need, how it should validate that information, and what results it should produce.

The main input should be a CSV file containing Date, Description, Category, and Amount, with one transaction per row. Dates will need to follow the YYYY-MM-DD format. Positive amounts should represent money coming in, while negative amounts will represent money going out. The program should accept amounts containing a $ symbol and give clear labels to blank descriptions or categories. If a date or amount is invalid, it should stop the upload and explain what needs correcting.

For the savings planner, users will enter their goal amount, the amount already saved, the weeks available, and how much they can put aside each week. The goal must be greater than zero, and the amount already saved must be non-negative, both with no more than two decimal places. Weeks must be a positive whole number, and the weekly amount available must be non-negative.

The chat input will accept non-empty questions about the uploaded data or savings plan. Gemini will use a GOOGLE_API_KEY read from Colab Secrets or a local environment variable and kept secret.

The planned outputs are a spending summary, a category table, and a savings plan showing the required weekly amount and any shortfall. The chat feature will make real Gemini requests using the current summary and savings plan as context.

I also identified limits in how the results should be presented. Money coming in may include refunds or transfers. Net cash flow will mean money in minus money out during the uploaded period; it will not represent the user’s account balance or guaranteed future savings. The program will also avoid estimating a monthly rate from an incomplete transaction export.

## 2nd October 2026 — Working through an example to help create pseudo code

for the past few days I worked through an example by hand to check the calculations before writing the pseudo code and finally the Python code. I used wages of $1,000 and expenses of $400 for rent, $50 and $25 for groceries, and $25 for transport. This gave $1,000 coming in, $500 going out, and a net cash flow of $500. Grouping the expenses by category gave Housing $400, Food $75, and Transport $25.
I then checked a savings example with a $500 goal, $100 already saved, four weeks remaining, and $80 available each week. The remaining amount was $400, meaning the user would need to save $100 per week. This left a weekly shortfall of $20. At their current contribution, they would have $420 saved by the end of the four weeks.
I also considered rounding. If one cent remains and there are three weeks available, the required weekly contribution must round up to one cent because contributions cannot include fractions of a cent. This avoids showing a contribution that would leave the user short of their goal.
After checking these examples, I wrote pseudocode for the upload, savings planner, and chat features. For uploads, I planned checks for a non-empty CSV with the four required columns, valid dates, and valid amounts converted into integer cents. Blank descriptions and categories will receive clear labels. The program will calculate money in, money out, and spending totals using a category dictionary, then display the summary and sorted category totals. A successful upload will replace the previous summary and clear the old chat. Invalid input will produce a useful error message and clear stale results.
For the savings planner, I outlined validation and calculations for the remaining amount, required weekly contribution, shortfall, and projected total. Remaining amounts and shortfalls will have a minimum of zero, and the required weekly contribution will round upwards. Recalculating the plan will replace the current plan and clear the old chat. I will probably write this logic myself rather than use a financial library.
For chat, I planned to require a non-empty message and a successfully loaded summary. The program will read the API key privately and send Gemini the finance system instruction, recent conversation, current summary, and current savings plan using requests with a timeout. Successful replies will be added to the chat history. Failed requests will show a useful status message without presenting an error as a Gemini reply.

## 5th October 2026 — Creating parts of the python code

I started converting my plan into Python by adding a setup cell to install specific versions of pandas, requests, and Gradio. Pinning the versions will help keep the notebook’s dependencies consistent. I then added the imports needed for handling transaction data, making API requests, building the interface, and validating money values. The cells are arranged to run in order, with separate functions for different responsibilities so each part is easier to explain and test.
I wrote the money_to_cents() function to convert dollar amounts into integer cents. For example, an input of 45.50 becomes 4550 cents. I used Decimal to parse the input without introducing binary floating-point rounding errors. The function removes surrounding spaces and supports dollar signs, including negative amounts written as -$45.50.
I also added validation to reject invalid numbers, infinite values, and amounts containing fractions of a cent. Each error gives a clear message explaining what needs correcting. The function accepts a label so the message can identify the input that caused the problem.
Finally, I added the dollars() function to display cent values as dollar amounts with commas and two decimal places. These helper functions will let the analysis and savings planner calculate using integer cents while presenting results in a familiar money format.

## 6 October 2026 — Creating sample transaction files

I created two fictional CSV files to use while developing the program. The first contains the transactions from my hand-worked example, with $1,000 coming in and $500 going out. The second represents a tighter budget, with $800 coming in and $750 going out.

Both files use the required Date, Description, Category and Amount columns. Having two datasets gives me different expected results to compare and a way to check that the program uses newly uploaded data.

## 8 October 2026 — Loading and validating CSV files and Calculating and displaying spending summaries

I wrote the `load_transactions()` function to read a CSV and turn its rows into transaction dictionaries. I used pandas to read the values as text so that my money-validation function can check each amount before it is used.

I added checks for missing uploads, incorrect file types, empty files, missing columns and invalid dates. The function also checks each amount and gives blank descriptions and categories the labels “No description” and “Uncategorised”. An invalid row stops the upload with an explanation.

I wrote `summarise_transactions()` to calculate money in, money out and net cash flow using a loop. Positive amounts are added to money in. Negative amounts are converted to positive spending values and added to money out.

I used a dictionary to keep a running spending total for each category. The summary also records the transaction count and date range. I added `summary_text()` to display the results with the largest spending categories first. It shows up to ten categories individually and combines any remaining categories into one total.

## 11 October 2026 — Implementing the savings planner and Setting up Gemini and the finance persona

I converted the savings-plan pseudocode into `calculate_savings_plan()`. It validates the goal, amount already saved, weeks remaining and weekly contribution, then calculates the remaining amount, required weekly contribution, weekly shortfall and projected total.

The calculations use integer cents, with the required weekly amount rounded upwards. I also prevented negative remaining amounts when a goal has already been reached. For the $500 example, the function calculates $100 needed each week, a $20 weekly shortfall and a projected total of $420. I added `plan_text()` to explain these results to the user.

I added the Gemini connection using the requests library and set the model to `gemini-3.5-flash-lite`. I wrote a system instruction for Campus Budget Coach to give brief budgeting explanations based on the supplied summary and savings plan.

The instruction tells the model to quote the figures accurately, ask for clarification when needed and redirect unrelated questions. It also tells the model to treat transaction labels as data. I added `read_api_key()` to retrieve `GOOGLE_API_KEY` from colab secrets.

## 13 October 2026 — Building the Gradio interface, Connecting the functions to the interface, Building chat requests and handling failures

Building chat requests and handling failures

I wrote `get_gemini_reply()` to send the current spending summary, savings plan and question to Gemini. It also includes up to ten recent conversation messages so follow-up questions have context. I added a helper to handle the different text formats returned in Gradio chat history.

The request uses a 45-second timeout. I added messages for connection failures, exhausted quota, rejected requests and responses without readable text. These failures are reported separately from model replies, so the interface only adds an assistant message when Gemini returns a usable answer.

I wrote callback functions to connect the analysis, savings planner and chat to Gradio. These functions return the text, tables and stored values that the interface needs.

A successful analysis replaces the current summary and clears the previous conversation. An invalid upload clears the old summary and results so the chat cannot continue using outdated figures. Calculating a new savings plan also clears the conversation. For chat, I added checks for empty questions, messages longer than 2,000 characters and questions sent before a transaction summary has been loaded.

I built the interface with three tabs: Your spending, Savings goal and Talk to your coach. The spending tab contains the CSV upload, sample files, analysis button, summary and category table. The savings tab contains the four planning inputs and the calculated results.

The chat tab contains the conversation, question box, Send button and status message. I used `gr.State` to store the current summary and plan for each browser session. I also added guidance about the CSV format, the limits of cash-flow figures and the information sent to Gemini when a question is submitted.

## 15 October 2026 — Documenting the currency-handling bug and fix and Testing calculations and invalid inputs

I documented a problem in the inherited workshop approach, which compared raw Amount values with zero. Reproducing that comparison with `$1000.00` and `-$50.00` produces a TypeError because the values are strings.

The new loader handles these values by converting them into integer cents before any comparisons or calculations. I added a regression case expecting $1,000 coming in, $50 going out and $950 net cash flow. This checks that the new approach handles the input that caused the earlier comparison to fail.

I added assertions using the expected results from my hand-worked example and the tighter-budget dataset. The savings tests cover the normal example, an affordable contribution, goals already reached or exceeded, and rounding cases where the remaining amount does not divide evenly across the weeks.

I also included invalid money values, invalid week values, empty files, missing columns and impossible dates. Further checks cover blank labels, the currency regression and clearing old interface results. The notebook’s saved output reports that all these assertion cases passed. These checks cover the calculations and validation; Gemini’s wording still needs separate review.

## 16 October 2026 — Reviewing live replies and launching the app

I reviewed the three saved live Gemini replies. The budgeting reply used the expected $1,000 money in, $500 money out, $500 net cash flow and $20 weekly savings shortfall. The unrelated dragon-story request was redirected to budgeting, and “Can I afford it?” prompted a question about the intended purchase.

I also included an optional connection check that reports HTTP status codes without displaying credentials. The saved authenticated check returned HTTP 200. The final launch cell builds the Gradio app, and its saved output records a temporary public link.

I added notes explaining the main functions and how they connect. I also noted the submission checks: reviewing my explanations and diary dates, checking repository and ZIP contents, confirming collaborator access and following the Blackboard submission requirements.
