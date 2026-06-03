---
jupyter:
  jupytext:
    formats: ipynb,md
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.19.3
  kernelspec:
    display_name: Python 3
    name: python3
---

<!-- #region id="c91225e4" -->
## Análise Reutilizável com Função

Para facilitar a análise de múltiplos DataFrames, vou definir uma função que encapsula todas as etapas de inspeção e visualização realizadas anteriormente.
<!-- #endregion -->

```python id="29648e78"
import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd

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

    # 5. Análise de Datas (Inicio e Fim)
    if 'Inicio' in df.columns and pd.api.types.is_datetime64_any_dtype(df['Inicio']):
        print(f"\nLimite inferior das datas de Inicio no {df_name}: {df['Inicio'].min()}")
        print(f"Limite superior das datas de Inicio no {df_name}: {df['Inicio'].max()}")

        # 6. Janela de tempo e média de apontamentos
        time_window = df['Inicio'].max() - df['Inicio'].min()
        print(f"Janela de tempo no {df_name}: {time_window}")

        time_window_days = time_window.days + (1 if time_window.seconds > 0 else 0)
        if time_window_days == 0:
            time_window_days = 1
        print(f"Média de apontamentos por dia no {df_name}: {round(df.shape[0] / time_window_days, 2)}")

        time_window_hours = time_window_days * 24
        print(f"Média de apontamentos por hora no {df_name}: {round(df.shape[0] / time_window_hours, 2)}")
    else:
        print(f"\nO DataFrame {df_name} não possui uma coluna 'Inicio' ou ela não é do tipo datetime. Pulando análises de tempo.")

    # 7. Média de registros por equipamento (se 'Tag' existe)
    if 'Tag' in df.columns:
        registros_por_tag = df.groupby('Tag').size()
        media_registros_por_equipamento = registros_por_tag.mean()
        print(f"\nMédia de registros por equipamento no {df_name}: {media_registros_por_equipamento:.2f}")
    else:
        print(f"\nO DataFrame {df_name} não possui uma coluna 'Tag'. Pulando média de registros por equipamento.")

    # 8. Distribuição temporal dos registros
    if 'Inicio' in df.columns and pd.api.types.is_datetime64_any_dtype(df['Inicio']):
        print(f"\n### Distribuição Temporal dos Registros para {df_name}")

        # Create copies of the dataframe for plotting to avoid SettingWithCopyWarning
        df_plot = df.copy()
        df_plot['Dia'] = df_plot['Inicio'].dt.date
        df_plot['Hora'] = df_plot['Inicio'].dt.hour

        distribuicao_diaria = df_plot.groupby('Dia').size().reset_index(name='Volume')
        distribuicao_horaria = df_plot.groupby('Hora').size().reset_index(name='Volume')

        plt.figure(figsize=(15, 6))
        sns.lineplot(x='Dia', y='Volume', data=distribuicao_diaria)
        plt.title(f'Volume de Registros de Apontamentos por Dia - {df_name}')
        plt.xlabel('Dia')
        plt.ylabel('Volume de Registros')
        plt.xticks(rotation=45)
        plt.grid(True)
        plt.tight_layout()
        plt.show()

        plt.figure(figsize=(15, 6))
        sns.lineplot(x='Hora', y='Volume', data=distribuicao_horaria)
        plt.title(f'Volume de Registros de Apontamentos por Hora do Dia - {df_name}')
        plt.xlabel('Hora do Dia')
        plt.ylabel('Volume de Registros')
        plt.xticks(range(0, 24))
        plt.grid(True)
        plt.tight_layout()
        plt.show()

    else:
        print(f"\nO DataFrame {df_name} não possui uma coluna 'Inicio' ou ela não é do tipo datetime. Pulando plots de distribuição temporal.")

```

<!-- #region id="b51ad4b2" -->
## Executando a Análise para `df_ap` (Parquet)
<!-- #endregion -->

```python colab={"base_uri": "https://localhost:8080/", "height": 1000} id="4f7e0814" outputId="d560c753-8a34-4367-8816-15d072f10eb6"
parquet_file_path = 'drive/MyDrive/hackathon-vale/apontamentos/desenvolver_apontamentos.parquet'
df_ap = pd.read_parquet(parquet_file_path)


analyze_dataframe(df_ap.copy(), 'df_ap_parquet')
```

<!-- #region id="a4a4001a" -->
## Executando a Análise para `desenvolver_apontamentos.xlsx`
<!-- #endregion -->

```python colab={"base_uri": "https://localhost:8080/", "height": 1000} id="5ce4005f" outputId="75c4724b-314f-4461-97b2-7802a292653b"
excel_file_path = 'drive/MyDrive/hackathon-vale/apontamentos/desenvolver_apontamentos.xlsx'
df_ap_excel_new = pd.read_excel(excel_file_path)

analyze_dataframe(df_ap_excel_new.copy(), 'df_ap_excel')
```
