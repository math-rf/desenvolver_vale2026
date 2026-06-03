---
jupyter:
  jupytext:
    formats: ipynb,py:percent,md
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.19.3
  kernelspec:
    display_name: Python 3
    name: python3
---

```python id="nxR5zQ54ZuLy"
import matplotlib.pyplot as plt
import seaborn as sns

def analyze_dataframe(df, df_name):
    print(f"\n{'='*60}\nIniciando Análise para o DataFrame: {df_name}\n{'='*60}")

    # 1. Shape e dtypes
    print(f"\nShape da tabela de {df_name}: {df.shape}")
    print(f"Colunas da tabela de {df_name} e seus tipos:\n {df.dtypes}")

    # 2. Checando ocorrências de 'NULL'/'null' em colunas object
    print(f"\nVerificando 'NULL'/'null' strings em colunas de texto para {df_name}:")
    object_cols = df.select_dtypes(include='object').columns
    for col in object_cols:
        count_null_strings = df[col].isin(['NULL', 'null']).sum()
        print(f"Ocorrências da string 'NULL'/'null' em {col} (no {df_name}): {count_null_strings}")

    # 3. Nulos por coluna
    print(f"\nNúmero de nulos por coluna no {df_name}:\n {df.isnull().sum()}")

    # 4. Registros duplicados
    print(f"\nNúmero de registros duplicados na tabela de {df_name}: {df.duplicated().sum()}")

    # 5. Análise de Datas (Data_Evento)
    if 'Data_Evento' in df.columns and pd.api.types.is_datetime64_any_dtype(df['Data_Evento']):
        print(f"\nLimite inferior das datas de Data_Evento no {df_name}: {df['Data_Evento'].min()}")
        print(f"Limite superior das datas de Data_Evento no {df_name}: {df['Data_Evento'].max()}")

        # 6. Janela de tempo e média de telemetria
        time_window = df['Data_Evento'].max() - df['Data_Evento'].min()
        print(f"Janela de tempo no {df_name}: {time_window}")

        time_window_days = time_window.days + (1 if time_window.seconds > 0 else 0)
        if time_window_days == 0:
            time_window_days = 1
        print(f"Média de telemetria por dia no {df_name}: {round(df.shape[0] / time_window_days, 2)}")

        time_window_hours = time_window_days * 24
        print(f"Média de telemetria por hora no {df_name}: {round(df.shape[0] / time_window_hours, 2)}")
    else:
        print(f"\nO DataFrame {df_name} não possui uma coluna 'Data_Evento' ou ela não é do tipo datetime. Pulando análises de tempo.")

    # 7. Média de registros por equipamento (se 'TAG' existe)
    if 'TAG' in df.columns:
        registros_por_tag = df.groupby('TAG').size()
        media_registros_por_equipamento = registros_por_tag.mean()
        print(f"\nMédia de registros por equipamento no {df_name}: {media_registros_por_equipamento:.2f}")
    else:
        print(f"\nO DataFrame {df_name} não possui uma coluna 'TAG'. Pulando média de registros por equipamento.")

    # 8. Distribuição temporal dos registros
    if 'Data_Evento' in df.columns and pd.api.types.is_datetime64_any_dtype(df['Data_Evento']):
        print(f"\n### Distribuição Temporal dos Registros para {df_name}")

        df_plot = df.copy()
        df_plot['Dia'] = df_plot['Data_Evento'].dt.date
        df_plot['Hora'] = df_plot['Data_Evento'].dt.hour

        distribuicao_diaria = df_plot.groupby('Dia').size().reset_index(name='Volume')
        distribuicao_horaria = df_plot.groupby('Hora').size().reset_index(name='Volume')

        plt.figure(figsize=(15, 6))
        sns.lineplot(x='Dia', y='Volume', data=distribuicao_diaria)
        plt.title(f'Volume de Registros de Telemetria por Dia - {df_name}')
        plt.xlabel('Dia')
        plt.ylabel('Volume de Registros')
        plt.xticks(rotation=45)
        plt.grid(True)
        plt.tight_layout()
        plt.show()

        plt.figure(figsize=(15, 6))
        sns.lineplot(x='Hora', y='Volume', data=distribuicao_horaria)
        plt.title(f'Volume de Registros de Telemetria por Hora do Dia - {df_name}')
        plt.xlabel('Hora do Dia')
        plt.ylabel('Volume de Registros')
        plt.xticks(range(0, 24))
        plt.grid(True)
        plt.tight_layout()
        plt.show()

    else:
        print(f"\nO DataFrame {df_name} não possui uma coluna 'Data_Evento' ou ela não é do tipo datetime. Pulando plots de distribuição temporal.")

```

```python id="I-SqwMQZcrUH"
def optimize_dataframe_types(df):

    print(f"\n{'='*60}\nOtimizando tipos de dados para o DataFrame\n{'='*60}")
    start_mem = df.memory_usage(deep=True).sum() / 1024**2
    print(f"Uso de memória inicial: {start_mem:.2f} MB")

    for col in df.columns:
        col_type = df[col].dtype

        if str(col_type).startswith('int'):
            # Downcast integer types
            if df[col].min() >= 0:
                # Unsigned integers
                if df[col].max() < 2**8: # 256
                    df[col] = df[col].astype('uint8')
                elif df[col].max() < 2**16: # 65536
                    df[col] = df[col].astype('uint16')
                elif df[col].max() < 2**32: # 4.29e9
                    df[col] = df[col].astype('uint32')
            else:
                # Signed integers
                if df[col].min() > -2**7 and df[col].max() < 2**7 - 1: # -128 to 127
                    df[col] = df[col].astype('int8')
                elif df[col].min() > -2**15 and df[col].max() < 2**15 - 1: # -32768 to 32767
                    df[col] = df[col].astype('int16')
                elif df[col].min() > -2**31 and df[col].max() < 2**31 - 1: # -2.14e9 to 2.14e9
                    df[col] = df[col].astype('int32')
        elif str(col_type).startswith('float'):
            # Downcast float types
            df[col] = df[col].astype('float32')
        elif col_type == 'object':
            # Convert object to category if appropriate
            num_unique_values = len(df[col].unique())
            num_total_values = len(df[col])
            if num_unique_values / num_total_values < 0.5:  # Heuristic: if less than 50% unique values
                df[col] = df[col].astype('category')

    end_mem = df.memory_usage(deep=True).sum() / 1024**2
    print(f"Uso de memória final: {end_mem:.2f} MB")
    print(f"Memória economizada: {(start_mem - end_mem):.2f} MB ({((start_mem - end_mem) / start_mem * 100):.2f}%)")
    print(f"{'='*60}\n")
    return df

output_dir = 'drive/MyDrive/hackathon-vale/pre-processado/telemetria/'

def process_telemetry_dataset(path, output_dir):
    df = None
    df_name = path.split('/')[-1]
    base_name, ext = os.path.splitext(df_name)
    output_path = os.path.join(output_dir, f"{base_name}_optimized.parquet")

    print(f"\nProcessing file: {df_name}")

    try:
        if path.endswith('.parquet'):
            df = pd.read_parquet(path)
        elif path.endswith('.xlsx'):
            df = pd.read_excel(path)
        else:
            print(f"Skipping unknown file type for: {df_name}")
            return

        for col_name in ['Data_Evento', 'Inicio_Turno', 'Fim_Turno']:
            if col_name in df.columns:
                try:
                    df[col_name] = pd.to_datetime(df[col_name], errors='coerce')
                    print(f"Converted '{col_name}' column to datetime for {df_name}.")
                except Exception as e:
                    print(f"Warning: Could not convert '{col_name}' column to datetime for {df_name}. Error: {e}")
            else:
                print(f"Column '{col_name}' not found in {df_name}. Skipping datetime conversion.")

        if 'Valor' in df.columns:
            try:
                df['Valor'] = pd.to_numeric(df['Valor'], errors='coerce')
                print(f"Converted 'Valor' column to numeric for {df_name}.")
            except Exception as e:
                print(f"Warning: Could not convert 'Valor' column to numeric for {df_name}. Error: {e}")
        else:
            print(f"Column 'Valor' not found in {df_name}. Skipping numeric conversion.")

        analyze_dataframe(df, df_name)
        print(f"Analyzed {df_name}.")

        df = optimize_dataframe_types(df)
        print(f"Optimized types for {df_name}.")

        os.makedirs(output_dir, exist_ok=True)
        df.to_parquet(output_path, index=False)
        print(f"Optimized DataFrame saved to: {output_path}")

    except Exception as e:
        print(f"Error processing {df_name}: {e}")

    finally:
        if df is not None:
            del df
            gc.collect()
            print(f"Finished processing and cleared memory for: {df_name}")
```

<!-- #region id="378b3b69" -->
### Processando `desenvolver_dontgo.xlsx`
<!-- #endregion -->

```python colab={"base_uri": "https://localhost:8080/", "height": 1000} id="0f055b99" outputId="70e4a9ec-9f0a-47aa-c51c-c3e64dbe1e2b"
process_telemetry_dataset(
    '/content/drive/MyDrive/hackathon-vale/telemetria/desenvolver_dontgo.xlsx', output_dir=output_dir)
```

<!-- #region id="8bd4d871" -->
### Processando `telemetry_jan.parquet`
<!-- #endregion -->

```python colab={"base_uri": "https://localhost:8080/", "height": 1000} id="9e1c8c27" outputId="a9c48e67-e423-45f1-c6bb-b4947bc10418"
process_telemetry_dataset('/content/drive/MyDrive/hackathon-vale/telemetria/telemetry_jan.parquet', output_dir=output_dir)
```

<!-- #region id="106cc7b6" -->
### Processando `telemetry_feb.parquet`
<!-- #endregion -->

```python colab={"base_uri": "https://localhost:8080/", "height": 1000} id="a6e28bcd" outputId="0544557a-d67f-497f-87eb-3f954913fa4a"
process_telemetry_dataset('/content/drive/MyDrive/hackathon-vale/telemetria/telemetry_feb.parquet', output_dir=output_dir)
```

<!-- #region id="5618c474" -->
### Processando `telemetry_mar.parquet`
<!-- #endregion -->

```python colab={"base_uri": "https://localhost:8080/", "height": 1000} id="963b70ed" outputId="85b66780-7f60-44e2-c003-f5c3003d51bf"
process_telemetry_dataset('/content/drive/MyDrive/hackathon-vale/telemetria/telemetry_mar.parquet', output_dir=output_dir)
```

<!-- #region id="f9b16369" -->
### Processando `telemetry_abr.parquet`
<!-- #endregion -->

```python colab={"base_uri": "https://localhost:8080/", "height": 1000} id="4a4d9e86" outputId="2338cbd5-29c7-4d7f-e29b-260b89ef8c8b"
process_telemetry_dataset('/content/drive/MyDrive/hackathon-vale/telemetria/telemetry_abr.parquet', output_dir=output_dir)
```

<!-- #region id="551147ad" -->
### Processando `telemetry_may.parquet`
<!-- #endregion -->

```python colab={"base_uri": "https://localhost:8080/", "height": 1000} id="fa07245b" outputId="48b01b06-50f7-4819-c87c-9a121a478c0b"
process_telemetry_dataset('/content/drive/MyDrive/hackathon-vale/telemetria/telemetry_may.parquet', output_dir=output_dir)
```

<!-- #region id="9827abce" -->
### Processando `telemetry_jun.parquet`
<!-- #endregion -->

```python colab={"base_uri": "https://localhost:8080/", "height": 1000} id="ff43c036" outputId="b9730259-e452-4f3d-c096-abb10ab5d251"
process_telemetry_dataset('/content/drive/MyDrive/hackathon-vale/telemetria/telemetry_jun.parquet', output_dir=output_dir)
```

<!-- #region id="ed3d21ba" -->
### Concatenando todos os DataFrames Otimizados
<!-- #endregion -->

```python colab={"base_uri": "https://localhost:8080/", "height": 882} id="d8e35c99" outputId="9d8d5286-bd68-4015-efb0-d12039d1c0e2"
import pandas as pd
import os
import gc

output_dir = 'drive/MyDrive/hackathon-vale/pre-processado/telemetria/'
final_telemetry_df = None

parquet_files = [os.path.join(output_dir, f) for f in os.listdir(output_dir) if f.endswith('_optimized.parquet')]

if not parquet_files:
    print("No optimized parquet files found in the output directory.")
else:
    print(f"Found {len(parquet_files)} optimized parquet files. Starting sequential concatenation...")

    for i, file_path in enumerate(parquet_files):
        print(f"Reading and concatenating file {i+1}/{len(parquet_files)}: {file_path}...")
        try:
            df_temp = pd.read_parquet(file_path)
            if final_telemetry_df is None:
                final_telemetry_df = df_temp
            else:
                final_telemetry_df = pd.concat([final_telemetry_df, df_temp], ignore_index=True)

            del df_temp
            gc.collect()
            print(f"Current final DataFrame shape: {final_telemetry_df.shape}")

        except Exception as e:
            print(f"Error processing {file_path}: {e}")

    if final_telemetry_df is not None:
        print("\nAll optimized DataFrames concatenated successfully.")
        print(f"Final DataFrame shape: {final_telemetry_df.shape}")
        display(final_telemetry_df.head())

        final_output_path = os.path.join(output_dir, 'final_telemetry.parquet')
        final_telemetry_df.to_parquet(final_output_path, index=False)
        print(f"Final concatenated DataFrame saved to: {final_output_path}")

        del final_telemetry_df
        gc.collect()
        print("Final DataFrame cleared from memory.")
    else:
        print("No DataFrames were successfully concatenated.")

```
