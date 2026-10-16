# ISYS2001-YoussefAli
Assessments
# Campus Budget Coach

A student finance assistant with CSV spending analysis, a custom savings planner and Gemini conversation grounded in the current results.

## Run in Colab

1. Open `finance_assistant.ipynb` in Google Colab, or upload the notebook through **File → Upload notebook**.
2. In Colab Secrets, enable Notebook access for `GOOGLE_API_KEY`.
3. Run the notebook from top to bottom. The first code cell installs the three pinned dependencies. A later cell creates both sample CSVs automatically.
4. Read the assertion results. The optional live-check cell makes real Gemini requests only when a key is available.
5. Run the final launch cell. In **Your spending**, click a sample file and **Analyse transactions**. Then try the savings tab and chat tab.

Try the first sample: money in **$1,000**, money out **$500**, net **$500**. The second sample changes those totals to **$800**, **$750** and **$50**. For a $500 goal, $100 already saved, four weeks and $80 available per week, the planner reports **$100 needed per week** and a **$20 weekly shortfall**.

## Your own CSV

Use `Date,Description,Category,Amount` headers. Dates use `YYYY-MM-DD`. Money in is positive; spending is negative. Amounts can be plain numbers or dollar-formatted values such as `-$45.50`, with no thousands separators. The app validates the complete file and reports invalid input instead of silently dropping rows. Blank categories become `Uncategorised`.

Only the financial summary, savings plan and recent messages are sent to Gemini when you send a question. The uploaded CSV is processed by the running app. Choose what financial information you are comfortable using. In Colab, Gradio may create a temporary share link; stop the runtime after use.

## Understanding the code

The notebook presents the six steps before the interface. It uses pandas to read CSVs, lists and dictionaries to store records, explicit loops for category totals, `requests.post` for Gemini, Gradio for the interface, and assertions for tests. The savings tool uses integer cents and arithmetic to calculate the remaining amount, weekly contribution, shortfall and projected total.

`Decimal` is Python's standard decimal-number type. It converts an input such as `45.50` into exactly 4,550 cents. The planner's formula `(remaining + weeks - 1) // weeks` divides integer cents and rounds up. Read and practise explaining these lines before submission.

The design prose and diary are an assisted draft. Review them and add your own explanation of decisions you understand or change. `DEVELOPER_DIARY.md` records only the actual date of work; the weekly schedule and deadline are on Blackboard.
