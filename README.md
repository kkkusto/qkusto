"""
Create a PowerPoint (PPTX) explaining Dynatrace Predictive Analysis + examples table.

Requirements:
  pip install python-pptx

Run:
  python dynatrace_predictive_analysis_ppt.py

Output:
  Dynatrace_Predictive_Analysis_Overview.pptx
"""

from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.enum.text import PP_ALIGN


def set_font(paragraph, size=24, bold=False):
    for run in paragraph.runs:
        run.font.size = Pt(size)
        run.font.bold = bold


def add_bullets_slide(prs, title, bullets, font_size=24):
    """Add a title + bullets slide."""
    layout = prs.slide_layouts[1]  # Title and Content
    slide = prs.slides.add_slide(layout)
    slide.shapes.title.text = title

    tf = slide.placeholders[1].text_frame
    tf.clear()

    for i, b in enumerate(bullets):
        if i == 0:
            p = tf.paragraphs[0]
        else:
            p = tf.add_paragraph()
        p.text = b
        p.level = 0
        p.font.size = Pt(font_size)

    return slide


def add_flow_slide(prs, title, steps, note="High-level flow"):
    """
    Create a simple visual flow with boxes and arrows.
    Avoids Mermaid; works in PPT itself.
    """
    slide = prs.slides.add_slide(prs.slide_layouts[5])  # Title Only
    slide.shapes.title.text = title

    # Layout params
    left = Inches(0.5)
    top = Inches(2.0)
    box_w = Inches(2.1)
    box_h = Inches(0.8)
    gap = Inches(0.25)

    # Add subtitle note
    tx = slide.shapes.add_textbox(Inches(0.5), Inches(1.25), Inches(12.5), Inches(0.4))
    p = tx.text_frame.paragraphs[0]
    p.text = note
    p.font.size = Pt(16)

    # Draw boxes + arrows
    shapes = slide.shapes
    for idx, step in enumerate(steps):
        x = left + idx * (box_w + gap)
        box = shapes.add_shape(
            autoshape_type_id=1,  # MSO_AUTO_SHAPE_TYPE.RECTANGLE
            left=x, top=top, width=box_w, height=box_h
        )
        box.text_frame.text = step
        box.text_frame.paragraphs[0].font.size = Pt(14)
        box.text_frame.paragraphs[0].alignment = PP_ALIGN.CENTER

        # Arrow between boxes
        if idx < len(steps) - 1:
            arrow = shapes.add_shape(
                autoshape_type_id=33,  # MSO_AUTO_SHAPE_TYPE.RIGHT_ARROW
                left=x + box_w, top=top + Inches(0.1),
                width=gap, height=box_h - Inches(0.2)
            )
            # No text in arrow

    return slide


def add_table_slide(prs, title, columns, rows, col_widths_inches=None, font_size=12):
    """Add a slide with a table."""
    slide = prs.slides.add_slide(prs.slide_layouts[5])  # Title Only
    slide.shapes.title.text = title

    # Table placement
    left = Inches(0.4)
    top = Inches(1.4)
    width = Inches(12.6)
    height = Inches(5.6)

    table_shape = slide.shapes.add_table(rows=len(rows) + 1, cols=len(columns), left=left, top=top, width=width, height=height)
    table = table_shape.table

    # Column widths
    if col_widths_inches and len(col_widths_inches) == len(columns):
        for i, w in enumerate(col_widths_inches):
            table.columns[i].width = Inches(w)

    # Header
    for c, col_name in enumerate(columns):
        cell = table.cell(0, c)
        cell.text = col_name
        p = cell.text_frame.paragraphs[0]
        p.font.bold = True
        p.font.size = Pt(font_size)
        p.alignment = PP_ALIGN.CENTER

    # Body
    for r, row in enumerate(rows, start=1):
        for c, val in enumerate(row):
            cell = table.cell(r, c)
            cell.text = str(val)
            p = cell.text_frame.paragraphs[0]
            p.font.size = Pt(font_size)
            p.alignment = PP_ALIGN.LEFT

    return slide


def build_ppt(output_path="Dynatrace_Predictive_Analysis_Overview.pptx"):
    prs = Presentation()

    # --- Title slide
    slide = prs.slides.add_slide(prs.slide_layouts[0])
    slide.shapes.title.text = "Dynatrace Predictive Analysis"
    slide.placeholders[1].text = "What it predicts • How it works • When it is used"

    # --- What / Why / How / Where (bullets)
    add_bullets_slide(prs, "What Dynatrace Predicts", [
        "Infrastructure capacity trends (CPU, memory, disk, network)",
        "Application & service performance trends (latency, errors, throughput)",
        "Traffic/workload growth patterns",
        "Resource exhaustion risk (e.g., running out of disk/CPU headroom)",
        "Early signals of abnormal behavior"
    ])

    add_bullets_slide(prs, "Why Predictive Analysis Matters", [
        "Prevent outages instead of reacting after failures",
        "Plan capacity with evidence (not guesses)",
        "Reduce firefighting and faster decision-making",
        "Improve reliability and customer experience"
    ])

    add_bullets_slide(prs, "How Dynatrace Predicts (High Level)", [
        "Collects telemetry (metrics, logs, traces)",
        "Learns what “normal” looks like from historical behavior",
        "Forecasts future values and highlights risk",
        "Shows predictions with confidence bands (range, not just one number)"
    ])

    add_bullets_slide(prs, "Where Predictions Are Used", [
        "Dashboards (forecast overlay on charts)",
        "Notebooks (ad-hoc analysis and forecasting)",
        "Alerts / early warnings (when a future threshold breach is likely)",
        "Automation workflows (optional actions like notify/scale/ticket)"
    ])

    # --- Simple flow slide
    add_flow_slide(
        prs,
        "Prediction Flow",
        steps=[
            "Collect\nHistory",
            "Learn\nNormal",
            "Forecast\nFuture",
            "Flag\nRisk",
            "Act:\nAlert/Auto"
        ],
        note="Overview: how predictive insights typically flow in Dynatrace"
    )

    # --- Examples table slide
    columns = ["#", "Scenario", "What Dynatrace Predicts", "Where It Appears", "Business Outcome"]
    rows = [
        [1, "Disk Capacity", "Disk free space < 5% in 4 days", "Dashboard, Alert", "Expand disk before outage"],
        [2, "CPU Saturation", "CPU ~95% during tomorrow peak", "Notebook, Workflow", "Scale early"],
        [3, "K8s Memory Limit", "Pod exceeds memory limit in 2 hours", "Workflow, Alert", "Increase limit; avoid crash"],
        [4, "API SLA Risk", "Checkout latency crosses SLA", "Dashboard Forecast", "Fix before release"],
        [5, "Traffic Growth", "Traffic +25% next month", "Notebook/Report", "Plan infra & budget"],
        [6, "DB Connections", "Connections hit max in 3 days", "Alert, Problems", "Increase pool; avoid failures"],
        [7, "Error Spike", "Error count spikes tonight", "Notebook/Dashboard", "Investigate early"],
        [8, "Predictive Scaling", "CPU > 80% in 30 minutes", "Workflow", "Auto-scale before impact"],
    ]
    add_table_slide(
        prs,
        "Examples (Practical)",
        columns=columns,
        rows=rows,
        col_widths_inches=[0.4, 2.0, 4.3, 2.3, 3.0],
        font_size=12
    )

    # --- Summary slide
    add_bullets_slide(prs, "Summary", [
        "Dynatrace can forecast future behavior for many metrics and trends",
        "It helps teams act before incidents happen",
        "Predictions are visible in dashboards/notebooks and can drive alerts & automation",
        "Best value comes from combining prediction + action (notify/scale/fix)"
    ])

    prs.save(output_path)
    print(f"✅ Created: {output_path}")


if __name__ == "__main__":
    build_ppt()
