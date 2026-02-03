"""
Create an Excel file with:
1) Sheet: AI_Perf_Test_UseCases (all use cases)
2) Sheet: Top5_UseCases (top 5 + Detailed Steps to Implement)

Requirements:
  pip install openpyxl
"""

from openpyxl import Workbook
from openpyxl.styles import Font, Alignment
from openpyxl.utils import get_column_letter


def style_header(ws, header_row=1):
    """Bold header + wrap + freeze top row."""
    for cell in ws[header_row]:
        cell.font = Font(bold=True)
        cell.alignment = Alignment(wrap_text=True, vertical="top")
    ws.freeze_panes = "A2"


def auto_fit_columns(ws, max_width=60):
    """Auto-size columns by content length (capped)."""
    for col_idx in range(1, ws.max_column + 1):
        col_letter = get_column_letter(col_idx)
        max_len = 0
        for row in range(1, ws.max_row + 1):
            v = ws.cell(row=row, column=col_idx).value
            if v is None:
                continue
            v = str(v)
            max_len = max(max_len, len(v))
        ws.column_dimensions[col_letter].width = min(max_len + 2, max_width)


def wrap_all_cells(ws):
    """Enable wrap text for all cells (useful for long steps)."""
    for row in ws.iter_rows(min_row=1, max_row=ws.max_row, min_col=1, max_col=ws.max_column):
        for cell in row:
            if cell.alignment is None:
                cell.alignment = Alignment(wrap_text=True, vertical="top")
            else:
                cell.alignment = Alignment(
                    wrap_text=True,
                    vertical="top",
                    horizontal=cell.alignment.horizontal
                )


def create_excel(output_path="AI_Performance_Testing_UseCases.xlsx"):
    wb = Workbook()

    # -------------------------
    # Sheet 1: All Use Cases
    # -------------------------
    ws1 = wb.active
    ws1.title = "AI_Perf_Test_UseCases"

    headers1 = ["Category", "AI Use Case", "Description", "Business Value"]
    ws1.append(headers1)

    all_use_cases = [
        ["Workload Modeling", "AI-Generated Load Profiles", "Creates realistic traffic based on production behavior", "Accurate performance validation"],
        ["Workload Modeling", "Persona-Based Load", "Creates user personas with different traffic patterns", "Find hidden bottlenecks"],

        ["Test Design", "Auto Script Generation", "Creates JMeter/k6 scripts from API specs", "Reduce scripting effort"],
        ["Test Design", "Intelligent Parameterization", "Identifies fields needing uniqueness", "Avoid false results"],

        ["Baselining", "Self-Learning Baselines", "Learns normal latency and CPU ranges", "No manual thresholds"],
        ["Baselining", "Adaptive SLA Tuning", "Updates thresholds automatically", "Always-relevant SLAs"],

        ["Prediction", "Bottleneck Prediction", "Predicts risky services before test", "Shift-left performance"],
        ["Prediction", "Capacity Forecasting", "Predict infra needed for growth", "Avoid outages"],

        ["Execution", "Smart Ramp-Up", "Adjusts load in real time", "Safer tests"],
        ["Execution", "Auto Test Selection", "Chooses which perf tests to run", "Faster CI/CD"],

        ["Monitoring", "Anomaly Detection", "Finds abnormal patterns", "Early issue detection"],
        ["Monitoring", "Early Abort", "Stops tests when failure predicted", "Save cost"],

        ["RCA", "Automated Root Cause", "Correlates logs, metrics, traces", "Fast troubleshooting"],
        ["RCA", "Log + Metric Fusion", "Summarizes issue", "Less manual analysis"],

        ["Optimization", "Code Optimization Suggestions", "Suggests caching, async, indexing", "Faster fixes"],
        ["Optimization", "Infra Optimization", "Suggests scaling & tuning", "Cost + performance balance"],

        ["Test Data", "Synthetic Data Generation", "Generates large realistic datasets", "Scalable testing"],
        ["Test Data", "Data Drift Detection", "Detects mismatch vs prod data", "Realistic tests"],

        ["Reporting", "AI Test Summary", "Creates executive-ready report", "Better communication"],
        ["Reporting", "NLQ Querying", "Ask questions in plain English", "Self-service insights"],
    ]

    for row in all_use_cases:
        ws1.append(row)

    style_header(ws1)
    wrap_all_cells(ws1)
    auto_fit_columns(ws1)

    # -------------------------
    # Sheet 2: Top 5 Use Cases
    # -------------------------
    ws2 = wb.create_sheet(title="Top5_UseCases")

    headers2 = ["Rank", "Category", "AI Use Case", "Description", "Business Value", "Detailed Steps to Implement"]
    ws2.append(headers2)

    top5 = [
        [1, "Workload Modeling", "AI-Generated Load Profiles",
         "Creates realistic traffic based on production behavior",
         "High accuracy workload simulation",
         "\n".join([
             "1. Collect production access logs (API gateway/Ingress/ALB)",
             "2. Parse requests into sessions (user_id/correlation_id + time window)",
             "3. Extract distributions: arrival rate, think time, session length, endpoint mix",
             "4. Train clustering/modeling (e.g., KMeans/DBSCAN) to learn personas & curves",
             "5. Generate workload model (time-varying RPS + scenario mix) and feed into JMeter/k6"
         ])],

        [2, "Monitoring", "Anomaly Detection",
         "Detects abnormal latency, CPU, memory patterns",
         "Early issue detection",
         "\n".join([
             "1. Export metrics to Prometheus/Datadog (latency p50/p95/p99, CPU, mem, errors)",
             "2. Build baseline window (e.g., last 14 stable runs) per endpoint/service",
             "3. Train anomaly model (IsolationForest / robust z-score / Prophet) per signal",
             "4. Score streaming metrics during test; detect multivariate anomalies",
             "5. Alert + tag timeframe (ramp stage, endpoint, pod, dependency) for quick triage"
         ])],

        [3, "RCA", "Automated Root Cause Analysis",
         "Correlates logs, metrics, traces",
         "Fast troubleshooting",
         "\n".join([
             "1. Ingest logs + traces + metrics (OpenTelemetry + log pipeline + time sync)",
             "2. Normalize identifiers (trace_id, span_id, request_id, pod/service name)",
             "3. Build dependency graph from traces (service-to-service edges + latency contribution)",
             "4. Run correlation on incident window (change-point + dependency attribution)",
             "5. Output top suspected causes (e.g., DB lock, downstream latency, CPU throttling) with evidence"
         ])],

        [4, "Test Design", "Auto Script Generation",
         "Generates performance scripts from APIs",
         "Reduce scripting effort",
         "\n".join([
             "1. Parse OpenAPI/Swagger or Postman collection",
             "2. Identify critical flows (auth → search → add → checkout) + required headers/tokens",
             "3. Generate request templates (payloads, query params) and correlation extractors",
             "4. Add parameterization rules (unique IDs, randomization, data pools) + assertions",
             "5. Export runnable scripts (JMeter JMX / k6 JS / Gatling Scala) + CI command"
         ])],

        [5, "Prediction", "Capacity Forecasting",
         "Predicts infra needed for growth",
         "Avoid outages",
         "\n".join([
             "1. Collect historical load vs resource usage per service (RPS ↔ CPU/mem/DB)",
             "2. Fit model (linear/elastic net/XGBoost) per service + key dependencies",
             "3. Simulate growth scenarios (peak hour + promo event + retries + cache cold start)",
             "4. Predict saturation points (CPU throttling, DB connections, queue depth)",
             "5. Recommend scaling plan (HPA targets, pod requests/limits, DB pool sizing) and validate with a test run"
         ])],
    ]

    for row in top5:
        ws2.append(row)

    style_header(ws2)
    wrap_all_cells(ws2)
    auto_fit_columns(ws2, max_width=70)

    # Save
    wb.save(output_path)
    print(f"✅ Excel created: {output_path}")


if __name__ == "__main__":
    # Change output path if you want
    create_excel("AI_Performance_Testing_UseCases.xlsx")
