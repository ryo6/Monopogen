graph LR
    %% --- クラス定義 (色とスタイルの設定) ---
    classDef structure fill:#f9f,stroke:#333,stroke-width:2px,color:black;
    classDef cell fill:#ff9,stroke:#333,stroke-width:2px,color:black;
    classDef cytokine fill:#9cf,stroke:#333,stroke-width:2px,color:black;
    classDef mediator fill:#c9f,stroke:#333,stroke-width:2px,color:black;
    classDef drug fill:#b8f,stroke:#333,stroke-width:2px,stroke-dasharray: 5 5,color:black;
    classDef score fill:#9f9,stroke:#333,stroke-width:2px,color:black;
    
    %% リンクスタイル用クラス
    classDef promote stroke:blue,stroke-width:2px;
    classDef inhibit stroke:red,stroke-width:2px,stroke-dasharray: 5 5;

    %% --- ノード配置 ---

    subgraph Inputs [入力とトリガー]
        Pathogens(病原体 c1):::structure
        subgraph Drugs [治療介入 de]
            d_IL4(Anti-IL-4):::drug
            d_IL13(Anti-IL-13):::drug
            d_IL17(Anti-IL-17):::drug
            d_IL22(Anti-IL-22):::drug
            d_IL31(Anti-IL-31):::drug
            d_TSLP(Anti-TSLP):::drug
            d_OX40L(Anti-OX40L):::drug
            d_IFNg(rIFNγ):::drug
        end
    end

    subgraph ImmuneResponse [免疫応答]
        subgraph Upstream [上流メディエーター]
            TSLP(TSLP c12):::cytokine
            OX40L(OX40L c13):::mediator
            FactorX("Factor X c15"):::mediator
        end
        
        subgraph ThCells [Th細胞]
            Th1(Th1 c2):::cell
            Th2(Th2 c3):::cell
            Th17(Th17 c4):::cell
            Th22(Th22 c5):::cell
        end

        subgraph Cytokines [サイトカインとケモカイン]
            IFNg("IFNγ c11"):::cytokine
            IL4("IL-4 c6"):::cytokine
            IL13("IL-13 c7"):::cytokine
            IL31("IL-31 c10"):::cytokine
            IL17("IL-17 c8"):::cytokine
            IL22("IL-22 c9"):::cytokine
            TARC("TARC c14"):::mediator
            Suppressor("Suppressor Y c16"):::mediator
        end
    end

    subgraph Outcomes [組織の状態と結果]
        SB("皮膚バリア機能 c0"):::structure
        EASI("EASIスコア"):::score
    end

    %% --- 接続の定義 ---

    %% 皮膚バリアと病原体の相互作用
    Pathogens ---|破壊| SB:::inhibit
    SB ---|侵入阻止| Pathogens:::inhibit
    
    %% 病原体による応答の誘導
    Pathogens -->|誘導| TSLP:::promote
    Pathogens -->|誘導| TARC:::promote
    Pathogens -->|分化誘導| Th1:::promote
    Pathogens -->|分化誘導| Th2:::promote
    Pathogens -->|分化誘導| Th17:::promote
    Pathogens -->|分化誘導| Th22:::promote
    Pathogens -.-|排除反応強化| Pathogens:::promote

    %% 上流メディエーターの作用
    TSLP -->|誘導| OX40L:::promote
    OX40L -.-|維持/アポトーシス抑制| Th1:::promote
    OX40L -.-|維持/アポトーシス抑制| Th2:::promote
    OX40L -.-|維持/アポトーシス抑制| Th17:::promote
    OX40L -.-|維持/アポトーシス抑制| Th22:::promote
    IL31 ---|生成阻害| FactorX:::inhibit
    FactorX -->|促進| TARC:::promote

    %% Th細胞の分化バイアスとサイトカイン産生
    IFNg -->|Th1分化促進| Th1:::promote
    IL4 -->|Th2分化促進| Th2:::promote
    TARC -->|Th2動員/分化促進| Th2:::promote
    
    Th1 --> IFNg:::promote
    Th2 --> IL4:::promote
    Th2 --> IL13:::promote
    Th2 --> IL31:::promote
    Th17 --> IL17:::promote
    Th22 --> IL22:::promote

    %% サイトカインの皮膚バリアへの影響
    IL22 -->|修復促進| SB:::promote
    IL22 ---|修復阻害| SB:::inhibit
    IL4 ---|修復阻害| SB:::inhibit
    IL13 ---|修復阻害| SB:::inhibit
    IL17 ---|修復阻害| SB:::inhibit
    IL31 ---|修復阻害 & 破壊| SB:::inhibit

    %% サイトカインの病原体排除への影響
    IFNg -.-|排除強化| Pathogens:::promote
    IL17 -.-|排除強化| Pathogens:::promote
    IL22 -.-|排除強化| Pathogens:::promote
    IL4 ---|排除阻害| Pathogens:::inhibit
    IL13 ---|排除阻害| Pathogens:::inhibit

    %% TARC制御ループ
    IL4 -->|誘導| TARC:::promote
    IFNg -->|誘導| TARC:::promote
    SB ---|生成抑制| TARC:::inhibit
    TARC -->|誘導| Suppressor:::promote
    Suppressor ---|生成抑制| TARC:::inhibit

    %% 薬剤効果 (de配列)
    d_IL4 ---|阻害| IL4:::inhibit
    d_IL13 ---|阻害| IL13:::inhibit
    d_IL17 ---|阻害| IL17:::inhibit
    d_IL22 ---|阻害| IL22:::inhibit
    d_IL31 ---|阻害| IL31:::inhibit
    d_TSLP ---|阻害| TSLP:::inhibit
    d_OX40L ---|阻害| OX40L:::inhibit
    d_IFNg -->|投与/増強| IFNg:::promote

    %% EASIスコア計算
    SB -.-> EASI
    Pathogens -.-> EASI
