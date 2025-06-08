# -
ご自身が開発したソースコード、スキルや成果について
# 力積値算出方法

## 概要
このコードは測定された圧力波のデータから、研究の対象となっている気泡径に絞った範囲でピーク圧力を検知し、シンプソン積分を行うことで力積値を算出するコードです。

## 主な機能
- (機能1)
- (機能2)
- (機能3)

## 使用技術
-Python

## 実際のコード
import numpy as np
import pandas as pd
from scipy.integrate import simpson
import glob
import os
# Excelファイルのパスをリスト化（フォルダ内のExcelファイルをすべて取得）
file_paths = glob.glob('/Users/12nak/pressure1.2l0.6/*.xlsx')  # .xlsx拡張子を指定

if not file_paths:
    print("指定されたフォルダにExcelファイルが見つかりません。処理を終了します。")
    exit()
# 結果を保存するリストを作成
results = []
total_files = len(file_paths)
print(f"{total_files} 件のExcelファイルが見つかりました。処理を開始します。\n")
# 各ファイルに対して処理を行う
for idx, file_path in enumerate(file_paths, 1):
    try:
        # 進捗を計算して表示
        progress = (idx / total_files) * 100
        print(f"[{progress:.2f}%] ファイル {file_path} を処理中... ({idx}/{total_files})")
        # Excelファイルを読み込む
        df = pd.read_excel(file_path, header=None)
    # 一列目を‐100から0.002ずつ増える500000行に置き換え
        new_column = np.linspace(-100, -100 + 0.002 * (500000 - 1), 500000)
        df.iloc[:500000, 0] = new_column
    # 327500行目以降で二列目の値が負の場合は0に置き換え
        df.iloc[327500:, 1] = df.iloc[327500:, 1].clip(lower=0)
    # 327500行目から342000行目の範囲で二列目の最大値を持つ行を特定
        data_to_search = df.iloc[327500:353000, 1]
        max_index = data_to_search.idxmax()
    # 最大値を持つ行から上に75行、下に200行の範囲を取得
        start_row = max(max_index - 75 , 0)  # 行番号が負になるのを防ぐ
        end_row = max_index + 200
        data_to_integrate = df.iloc[start_row:end_row, 1]
    # シンプソン法で積分（条件に合うデータがある場合のみ実行）
        if len(data_to_integrate) > 1:
           dx = 0.002  # データ間の間隔（サンプリング間隔）を指定
           integral_raw = simpson(data_to_integrate, dx=dx)
           min_value = data_to_integrate.min()
           offset = min_value * dx * len(data_to_integrate)
           integral_value = integral_raw - offset
        else:
           integral_value = None
    # 結果をリストに追加
        results.append({"File": file_path, "Integral Result": integral_value})
    except Exception as e:
        print(f"エラーが発生しました（ファイル: {file_path}）: {e}")
        results.append({"File": file_path, "Integral Result": None})
# 結果をDataFrameに変換
results_df = pd.DataFrame(results)
# 出力先ディレクトリとファイルパスを指定
output_dir = '/Users/12nak/pressure1.2l0.6'
output_path = os.path.join(output_dir, '力積解析results1.2l0.6ver.2.xlsx')
# 出力先ディレクトリが存在しない場合は作成
os.makedirs(output_dir, exist_ok=True)
# 結果を新しいExcelファイルに保存
results_df.to_excel(output_path, index=False)
print(f"全ファイルの処理が完了しました。結果は {output_path} に保存されました。")
