# AI-Code-Reviewer
import streamlit as st
import subprocess
import tempfile
import os
from radon.complexity import cc_visit

st.set_page_config(page_title="AI Code Reviewer", layout="wide")
st.title("🤖 AI Code Reviewer")
st.write("Paste or upload Python code to analyze style, formatting, and complexity.")


uploaded_file = st.file_uploader("Upload Python File", type=["py"])
code_input = st.text_area("Or Paste Python Code Here", height=300)


code = ""
if uploaded_file:
    code = uploaded_file.read().decode("utf-8")
elif code_input:
    code = code_input

if code:
    st.subheader("📄 Original Code")
    st.code(code, language="python")

    # Save to temp file
    with tempfile.NamedTemporaryFile(delete=False, suffix=".py") as temp_file:
        temp_file.write(code.encode("utf-8"))
        temp_path = temp_file.name

    # flake8 analysis
    st.subheader("🔍 flake8 Style Analysis")
    flake8_result = subprocess.run(
        ["flake8", temp_path],
        capture_output=True,
        text=True
    )
    flake8_output = flake8_result.stdout.strip() if flake8_result.stdout.strip() else "No style issues found."
    st.text(flake8_output)

    # black diff formatting
    st.subheader("🎨 black Formatting Suggestions")
    black_diff = subprocess.run(
        ["black", "--diff", temp_path],
        capture_output=True,
        text=True
    )
    black_output = black_diff.stdout.strip() if black_diff.stdout.strip() else "Code already well formatted."
    st.text(black_output)

    # Apply black formatting
    subprocess.run(["black", temp_path], capture_output=True, text=True)
    with open(temp_path, "r", encoding="utf-8") as file:
        formatted_code = file.read()

    st.subheader("✅ Improved Code")
    st.code(formatted_code, language="python")

    # Radon complexity
    st.subheader("📊 Complexity Analysis (Radon)")
    complexity_blocks = cc_visit(code)
    complexity_summary = ""

    if complexity_blocks:
        for block in complexity_blocks:
            result = (
                f"Function/Class: {block.name} | "
                f"Complexity: {block.complexity} | "
                f"Rank: {block.letter}"
            )
            st.write(result)
            complexity_summary += result + "\n"
    else:
        st.write("No complex functions or classes detected.")
        complexity_summary = "No complex functions or classes detected."

    # Final report
    report = f"""
AI CODE REVIEW REPORT
======================

FLAKE8 STYLE ISSUES:
{flake8_output}

BLACK FORMATTING SUGGESTIONS:
{black_output}

COMPLEXITY ANALYSIS:
{complexity_summary}

RECOMMENDATIONS:
- Fix style violations reported by flake8
- Apply black formatting consistently
- Reduce complexity in high-score functions
- Maintain PEP8 compliance
"""

    st.subheader("📝 Summary Report")
    st.text(report)

    
    st.download_button(
        label="📥 Download Analysis Report",
        data=report,
        file_name="analysis_report.txt",
        mime="text/plain"
    )

    
    os.unlink(temp_path)

else:
    st.info("Upload or paste Python code to begin analysis.")

