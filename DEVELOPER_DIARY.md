# Developer's diary

I am starting this diary today on the 28/08/2026 to start tracking my progress for this assignment and project.

## 28 August 2026 — Choose the problem and design the app

Choose the Problem: The problem i have chosen is that small purchases are hard to track alongside large bills. So a university student would need a student spending and saving coach to assist them. The student needs a clear spending summary and a realistic weekly savings amount, rather than a generic answer that ignores their transactions. 

**Objective:** build a student spending and savings coach using the techniques in the unit.

**Decisions recorded before implementation:** im recording my decisions before implementation before hand to keep the problem narrow; i need to represent money as integer cents; describe the upload as cash flow rather than assume it is a complete monthly budget; use a custom savings calculation; keep API keys in Colab Secrets; and use requests for the Gemini call as required by the brief.

**What was checked:** im looking back at my existing Module 07 and 08 work, and the Module 09, 10 and 11 course examples provided in the unit repo given to us to read for how to use Gradio, error handling and assertion patterns

**Next:**  Identify inputs and outputs to help later on.

## 29 August 2026 — Choosing my inputs and outputs

I worked on defining the inputs and outputs for the finance assistant before starting development. This will help me work out what information the program should need, how it should validate that information, and what results it should produce.

The main input should be a CSV file containing Date, Description, Category, and Amount, with one transaction per row. Dates will need to follow the YYYY-MM-DD format. Positive amounts should represent money coming in, while negative amounts will represent money going out. The program should accept amounts containing a $ symbol and give clear labels to blank descriptions or categories. If a date or amount is invalid, it should stop the upload and explain what needs correcting.

For the savings planner, users will enter their goal amount, the amount already saved, the weeks available, and how much they can put aside each week. The goal must be greater than zero, and the amount already saved must be non-negative, both with no more than two decimal places. Weeks must be a positive whole number, and the weekly amount available must be non-negative.

The chat input will accept non-empty questions about the uploaded data or savings plan. Gemini will use a GOOGLE_API_KEY read from Colab Secrets or a local environment variable and kept secret.

The planned outputs are a spending summary, a category table, and a savings plan showing the required weekly amount and any shortfall. The chat feature will make real Gemini requests using the current summary and savings plan as context.

I also identified limits in how the results should be presented. Money coming in may include refunds or transfers. Net cash flow will mean money in minus money out during the uploaded period; it will not represent the user’s account balance or guaranteed future savings. The program will also avoid estimating a monthly rate from an incomplete transaction export.
