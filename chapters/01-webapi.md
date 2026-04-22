# 第1章：WebAPIの基本

> 執筆者：原 翔悟<br>
> 最終更新：2026-04-22

## この章で学ぶこと

（この章で扱うトピックの概要を2〜3行で書く。自分の言葉で。）

例：この章では、インターネット上のサービス（API）からデータを取得して、アプリ内に表示する方法を学ぶ。具体的にはiTunes Search APIを使って音楽を検索し、その結果をリスト表示するアプリを題材にする。

## 模範コードの全体像

（教員から配布された模範コードをここに貼り付ける）

```swift
import SwiftUI

// MARK: - データモデル

struct SearchResponse: Codable {
   let results: [Song]
}

struct Song: Codable, Identifiable {
   let trackId: Int
   let trackName: String
   let artistName: String
   let artworkUrl100: String
   let previewUrl: String?
   
   var id: Int { trackId }
}

// MARK: - メインビュー

struct ContentView: View {
   @State private var songs: [Song] = []
   @State private var searchText: String = ""
   @State private var isLoading: Bool = false
   
   var body: some View {
      NavigationStack {
         VStack {
            // 検索バー
            HStack {
               TextField("アーティスト名を入力", text: $searchText)
                  .textFieldStyle(.roundedBorder)
               
               Button("検索") {
                  Task {
                     await searchMusic()
                  }
               }
               .buttonStyle(.borderedProminent)
               .disabled(searchText.isEmpty)
            }
            .padding(.horizontal)
            
            // 検索結果リスト
            if isLoading {
               ProgressView("検索中...")
                  .padding()
               Spacer()
            } else if songs.isEmpty {
               ContentUnavailableView(
                  "曲を検索してみよう",
                  systemImage: "music.note",
                  description: Text("アーティスト名を入力して検索ボタンを押してください")
               )
            } else {
               List(songs) { song in
                  SongRow(song: song)
               }
            }
         }
         .navigationTitle("Music Search")
      }
   }
   
   // MARK: - API通信
   
   func searchMusic() async {
      guard let encodedText = searchText.addingPercentEncoding(
         withAllowedCharacters: .urlQueryAllowed
      ) else { return }
      
      let urlString = "https://itunes.apple.com/search?term=\(encodedText)&media=music&country=jp&limit=25"
      
      guard let url = URL(string: urlString) else { return }
      
      isLoading = true
      
      do {
         let (data, _) = try await URLSession.shared.data(from: url)
         let response = try JSONDecoder().decode(SearchResponse.self, from: data)
         songs = response.results
      } catch {
         print("エラー: \(error.localizedDescription)")
         songs = []
      }
      
      isLoading = false
   }
}

// MARK: - 曲の行ビュー

struct SongRow: View {
   let song: Song
   
   var body: some View {
      HStack(spacing: 12) {
         AsyncImage(url: URL(string: song.artworkUrl100)) { image in
            image
               .resizable()
               .aspectRatio(contentMode: .fill)
         } placeholder: {
            Color.gray.opacity(0.3)
         }
         .frame(width: 60, height: 60)
         .clipShape(RoundedRectangle(cornerRadius: 8))
         
         VStack(alignment: .leading, spacing: 4) {
            Text(song.trackName)
               .font(.headline)
               .lineLimit(1)
            
            Text(song.artistName)
               .font(.subheadline)
               .foregroundStyle(.secondary)
         }
      }
      .padding(.vertical, 4)
   }
}

#Preview {
   ContentView()
}
```

**このアプリは何をするものか：**

iTunes Search APIを使い、音楽（曲）を検索して結果をリスト表示するもの。

## コードの詳細解説

### データモデル（Codable構造体）

```swift
struct SearchResponse: Codable {
   let results: [Song]
}

struct Song: Codable, Identifiable {
   let trackId: Int
   let trackName: String
   let artistName: String
   let artworkUrl100: String
   let previewUrl: String?
   
   var id: Int { trackId }
}
```

**何をしているか：**
JSONからデータを構造体に変換して、保持する際に使うデータ構造を定義している。

**なぜこう書くのか：**
（別の書き方ではなく、この書き方が選ばれている理由を説明する）

**もしこう書かなかったら：**
（この部分を省略したり変えたりすると何が起きるか。実際に試した結果があればここに書く）

---

### API通信の処理

```swift
func searchMusic() async {
      guard let encodedText = searchText.addingPercentEncoding(
         withAllowedCharacters: .urlQueryAllowed
      ) else { return }
      
      let urlString = "https://itunes.apple.com/search?term=\(encodedText)&media=music&country=jp&limit=25"
      
      guard let url = URL(string: urlString) else { return }
      
      isLoading = true
      
      do {
         let (data, _) = try await URLSession.shared.data(from: url)
         let response = try JSONDecoder().decode(SearchResponse.self, from: data)
         songs = response.results
      } catch {
         print("エラー: \(error.localizedDescription)")
         songs = []
      }
      
      isLoading = false
   }
```

**何をしているか：**
検索欄に入力された文字をパーセントエンコーディング（URLで使用できない文字（日本語、スペース、一部の記号）を、「%」と2桁の16進数に変換）している。<br>
guard letで正常に変数を定義できたら、次へ。<br>
URLを文字列で組み立てて、その文字列からURL型に変数を定義。<br>
guard letで正常に変数を定義できたら、次へ。<br>
URLから非同期で通信してデータを取得する。<br>
取得したデータから、構造体「SearchResponse」に当てはめて、JSONとしてデコードする。<br>

**なぜこう書くのか：**

**もしこう書かなかったら：**

---

### ビューの構成

```swift

struct ContentView: View {
   @State private var songs: [Song] = []
   @State private var searchText: String = ""
   @State private var isLoading: Bool = false
   
   var body: some View {
      NavigationStack {
         VStack {
            // 検索バー
            HStack {
               TextField("アーティスト名を入力", text: $searchText)
                  .textFieldStyle(.roundedBorder)
               
               Button("検索") {
                  Task {
                     await searchMusic()
                  }
               }
               .buttonStyle(.borderedProminent)
               .disabled(searchText.isEmpty)
            }
            .padding(.horizontal)
            
            // 検索結果リスト
            if isLoading {
               ProgressView("検索中...")
                  .padding()
               Spacer()
            } else if songs.isEmpty {
               ContentUnavailableView(
                  "曲を検索してみよう",
                  systemImage: "music.note",
                  description: Text("アーティスト名を入力して検索ボタンを押してください")
               )
            } else {
               List(songs) { song in
                  SongRow(song: song)
               }
            }
         }
         .navigationTitle("Music Search")
      }
   }
}

struct SongRow: View {
   let song: Song
   
   var body: some View {
      HStack(spacing: 12) {
         AsyncImage(url: URL(string: song.artworkUrl100)) { image in
            image
               .resizable()
               .aspectRatio(contentMode: .fill)
         } placeholder: {
            Color.gray.opacity(0.3)
         }
         .frame(width: 60, height: 60)
         .clipShape(RoundedRectangle(cornerRadius: 8))
         
         VStack(alignment: .leading, spacing: 4) {
            Text(song.trackName)
               .font(.headline)
               .lineLimit(1)
            
            Text(song.artistName)
               .font(.subheadline)
               .foregroundStyle(.secondary)
         }
      }
      .padding(.vertical, 4)
   }
}
```

**何をしているか：**

**なぜこう書くのか：**

**もしこう書かなかったら：**

---

（必要に応じてセクションを増やす）

## 新しく学んだSwiftの文法・API

| 項目 | 説明 | 使用例 |
|------|------|--------|
| 例：`Codable` | JSONデータとSwiftの構造体を相互変換するプロトコル、Codable=Encodable+Decodable | `struct Song: Codable { ... }` |
| 例：`async/await` | 非同期処理を同期的に書ける構文 | `let data = try await URLSession.shared.data(from: url)` |
| `Identifiable` | そのデータが、他と区別できる一意のID（識別子）を持っていること | `struct Song: Codable, Identifiable { let trackId: Int}` |
| | | |
| | | |

## 自分の実験メモ

（模範コードを改変して試したことを書く）

**実験1：**
- やったこと：
- 結果：
- わかったこと：

**実験2：**
- やったこと：
- 結果：
- わかったこと：

## AIに聞いて特に理解が深まった質問 TOP3

1. **質問：**
   **得られた理解：**

2. **質問：**
   **得られた理解：**

3. **質問：**
   **得られた理解：**

## この章のまとめ

（この章で学んだ最も重要なことを、未来の自分が読み返したときに役立つように書く）
