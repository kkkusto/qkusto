import os
import pandas as pd
import plotly.express as px

# --- Configuration ---
input_folder = "path/to/your/folder"   # 🔸 change this to your folder path
output_folder = input_folder           # save in same folder or set a new path

# --- Iterate over CSV files ---
for file in os.listdir(input_folder):
    if file.endswith(".csv"):
        file_path = os.path.join(input_folder, file)
        df = pd.read_csv(file_path)

        # --- Expecting columns: Tool Name, App Count ---
        if not {"Tool Name", "App Count"}.issubset(df.columns):
            print(f"Skipping {file}: required columns missing.")
            continue

        # Create pivot table (Tool Name on Y-axis, single value heatmap)
        # If multiple entries per tool, take mean or sum
        data = df.groupby("Tool Name", as_index=False)["App Count"].sum()

        # Create interactive heatmap
        fig = px.imshow(
            [data["App Count"].values],
            labels=dict(x="Tool Name", color="App Count"),
            x=data["Tool Name"],
            color_continuous_scale="Viridis",
            aspect="auto"
        )
        fig.update_layout(
            title=f"Heatmap for {file}",
            xaxis=dict(side="bottom"),
            yaxis=dict(showticklabels=False),
        )

        # --- Save as HTML ---
        output_name = os.path.splitext(file)[0] + "_heatmap.html"
        output_path = os.path.join(output_folder, output_name)
        fig.write_html(output_path, include_plotlyjs="cdn", full_html=True)

        print(f"✅ Saved heatmap: {output_path}")
