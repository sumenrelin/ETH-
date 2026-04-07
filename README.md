# ETH-
ETHで増
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

/// @title Baseチェーン専用 今日の言葉人気コンテストコントラクト
/// @notice このコントラクトはBaseチェーン上で完全に新規に設計されたものです
contract 今日の言葉コンテスト {

    // 言葉の構造体定義
    struct 言葉 {
        string 内容;           // ユーザーが投稿した今日の言葉
        address 投稿者;         // 投稿者のアドレス
        uint256 人気度;         // ETHで増やされた人気度（重み）
        uint256 投稿時間;       // 投稿または最後に更新された時間
    }

    // 全ての言葉を保存する配列
    言葉[] public 全言葉リスト;

    // 各アドレスが最後に投稿した言葉のインデックスを記録
    mapping(address => uint256) public ユーザーの最終投稿インデックス;

    // イベント定義
    event 言葉が投稿された(uint256 インデックス, address 投稿者, string 内容, uint256 初期人気度);
    event 人気度が上昇した(uint256 インデックス, uint256 追加金額, uint256 新しい人気度);

    // コンストラクタ（何も初期化しない）
    constructor() {
        // デプロイ時に特別な初期化は行いません
    }

    /// @notice 今日の言葉を投稿または更新する関数。最低0.0001 ETHが必要です
    /// @param _内容 投稿する言葉の内容（最大280文字）
    function 今日の言葉を投稿する(string calldata _内容) external payable {
        require(bytes(_内容).length > 0 && bytes(_内容).length <= 280, "言葉の長さは1〜280文字の間でなければなりません");
        require(msg.value >= 0.0001 ether, "初期人気度として最低0.0001 ETHが必要です");

        uint256 現在のインデックス;

        if (ユーザーの最終投稿インデックス[msg.sender] == 0 && 全言葉リスト.length == 0) {
            // 最初の言葉を投稿
            現在のインデックス = 全言葉リスト.length;
            全言葉リスト.push(言葉({
                内容: _内容,
                投稿者: msg.sender,
                人気度: msg.value,
                投稿時間: block.timestamp
            }));
        } else if (ユーザーの最終投稿インデックス[msg.sender] > 0) {
            // 既存の言葉を更新
            現在のインデックス = ユーザーの最終投稿インデックス[msg.sender] - 1;
            全言葉リスト[現在のインデックス].内容 = _内容;
            全言葉リスト[現在のインデックス].人気度 += msg.value;
            全言葉リスト[現在のインデックス].投稿時間 = block.timestamp;
        } else {
            // 新規投稿
            現在のインデックス = 全言葉リスト.length;
            全言葉リスト.push(言葉({
                内容: _内容,
                投稿者: msg.sender,
                人気度: msg.value,
                投稿時間: block.timestamp
            }));
        }

        ユーザーの最終投稿インデックス[msg.sender] = 現在のインデックス + 1;

        emit 言葉が投稿された(現在のインデックス, msg.sender, _内容, msg.value);
    }

    /// @notice 特定の言葉に人気度（ETH）を追加する関数
    /// @param _インデックス 人気度を上げたい言葉のインデックス
    function 人気度を上げる(uint256 _インデックス) external payable {
        require(_インデックス < 全言葉リスト.length, "指定された言葉は存在しません");
        require(msg.value > 0, "人気度を上げるにはETHを送信する必要があります");

        全言葉リスト[_インデックス].人気度 += msg.value;
        全言葉リスト[_インデックス].投稿時間 = block.timestamp;

        emit 人気度が上昇した(_インデックス, msg.value, 全言葉リスト[_インデックス].人気度);
    }

    /// @notice 現在最も人気のある言葉を取得する
    /// @return 最も人気度の高い言葉の情報
    function 最も人気のある言葉を取得する() external view returns (言葉 memory) {
        require(全言葉リスト.length > 0, "まだ言葉が投稿されていません");

        uint256 最高インデックス = 0;
        uint256 最高人気度 = 全言葉リスト[0].人気度;

        for (uint256 i = 1; i < 全言葉リスト.length; i++) {
            if (全言葉リスト[i].人気度 > 最高人気度) {
                最高人気度 = 全言葉リスト[i].人気度;
                最高インデックス = i;
            }
        }

        return 全言葉リスト[最高インデックス];
    }

    /// @notice 指定したアドレスが最後に投稿した言葉を取得する
    /// @param _投稿者 確認したいユーザーのアドレス
    function ユーザーの最終言葉を取得する(address _投稿者) external view returns (言葉 memory) {
        uint256 インデックス = ユーザーの最終投稿インデックス[_投稿者];
        require(インデックス > 0, "このユーザーはまだ言葉を投稿していません");
        return 全言葉リスト[インデックス - 1];
    }

    /// @notice 現在投稿されている言葉の総数を取得する
    function 全言葉の数を取得する() external view returns (uint256) {
        return 全言葉リスト.length;
    }

    /// @notice コントラクトに蓄積されたETHを引き出す（簡易版）
    function 資金を引き出す() external {
        require(msg.sender == tx.origin, "直接呼び出したアドレスからのみ引き出せます");
        payable(msg.sender).transfer(address(this).balance);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

/// @title Baseチェーン専用 今日の言葉アリーナコントラクト
/// @notice このコントラクトはBaseチェーン上で完全に新規に設計されたオリジナルコントラクトです
contract DailyWordArena {

    // 言葉の構造体定義
    struct WordEntry {
        string content;           // ユーザーが投稿した今日の言葉
        address author;           // 投稿者のアドレス
        uint256 popularity;       // ETHで増やされた人気度
        uint256 timestamp;        // 投稿または最後に更新された時間
    }

    // 全ての言葉を保存する配列
    WordEntry[] public allEntries;

    // 各アドレスが最後に投稿した言葉のインデックスを記録
    mapping(address => uint256) public lastEntryIndex;

    // イベント定義
    event WordPosted(uint256 indexed index, address author, string content, uint256 initialPopularity);
    event PopularityIncreased(uint256 indexed index, uint256 addedAmount, uint256 newPopularity);

    // コンストラクタ（何も初期化しない）
    constructor() {
        // デプロイ時に特別な初期化は行いません
    }

    /// @notice 今日の言葉を投稿または更新する関数。最低0.0001 ETHが必要です
    /// @param _content 投稿する言葉の内容（最大280文字）
    function postWord(string calldata _content) external payable {
        require(bytes(_content).length > 0 && bytes(_content).length <= 280, unicode"言葉の長さは1〜280文字の間でなければなりません");
        require(msg.value >= 0.0001 ether, unicode"初期人気度として最低0.0001 ETHが必要です");

        uint256 currentIndex;

        if (lastEntryIndex[msg.sender] == 0) {
            // 新規投稿
            currentIndex = allEntries.length;
            allEntries.push(WordEntry({
                content: _content,
                author: msg.sender,
                popularity: msg.value,
                timestamp: block.timestamp
            }));
        } else {
            // 既存の言葉を更新
            currentIndex = lastEntryIndex[msg.sender] - 1;
            allEntries[currentIndex].content = _content;
            allEntries[currentIndex].popularity += msg.value;
            allEntries[currentIndex].timestamp = block.timestamp;
        }

        lastEntryIndex[msg.sender] = currentIndex + 1;

        emit WordPosted(currentIndex, msg.sender, _content, msg.value);
    }

    /// @notice 特定の言葉に人気度（ETH）を追加する関数
    /// @param _index 人気度を上げたい言葉のインデックス
    function increasePopularity(uint256 _index) external payable {
        require(_index < allEntries.length, unicode"指定された言葉は存在しません");
        require(msg.value > 0, unicode"人気度を上げるにはETHを送信する必要があります");

        allEntries[_index].popularity += msg.value;
        allEntries[_index].timestamp = block.timestamp;

        emit PopularityIncreased(_index, msg.value, allEntries[_index].popularity);
    }

    /// @notice 現在最も人気のある言葉を取得する
    /// @return 最も人気度の高い言葉の情報
    function getMostPopularWord() external view returns (WordEntry memory) {
        require(allEntries.length > 0, unicode"まだ言葉が投稿されていません");

        uint256 highestIndex = 0;
        uint256 highestPopularity = allEntries[0].popularity;

        for (uint256 i = 1; i < allEntries.length; i++) {
            if (allEntries[i].popularity > highestPopularity) {
                highestPopularity = allEntries[i].popularity;
                highestIndex = i;
            }
        }

        return allEntries[highestIndex];
    }

    /// @notice 指定したアドレスが最後に投稿した言葉を取得する
    /// @param _author 確認したいユーザーのアドレス
    function getLastWordOf(address _author) external view returns (WordEntry memory) {
        uint256 index = lastEntryIndex[_author];
        require(index > 0, unicode"このユーザーはまだ言葉を投稿していません");
        return allEntries[index - 1];
    }

    /// @notice 現在投稿されている言葉の総数を取得する
    function getTotalWords() external view returns (uint256) {
        return allEntries.length;
    }

    /// @notice コントラクトに蓄積されたETHを引き出す（簡易版）
    function withdrawFunds() external {
        require(msg.sender == tx.origin, unicode"直接呼び出したアドレスからのみ引き出せます");
        payable(msg.sender).transfer(address(this).balance);
    }
}
