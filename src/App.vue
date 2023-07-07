<!-- Board definitions.
    Copyright (C) 2023 Hideaki Sakai
    
    This file is part of Free-Shogi-Board-TS.
    
    Free-Shogi-Board is free software: you can redistribute it and/or modify
    it under the terms of the GNU Affero General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.
    
    Free-Shogi-Board is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU Affero General Public License for more details.
    
    You should have received a copy of the GNU Affero General Public License
    along with Free-Shogi-Board.  If not, see <https://www.gnu.org/licenses/>.
-->

<script setup lang="ts">
import KomaSymbols from './components/KomaSymbols.vue'
import { ref, onBeforeMount } from 'vue'
import type { Ref } from 'vue'

// マスを表わすクラス
class Masu {
    suji: number;
    dan: number;
    key: string;

    constructor(s: number, d: number) {
        this.suji = s;
        this.dan = d;
        this.key = `Masu-${s}${d}`;
    }

    getSuji() { return this.suji; }
    getDan() { return this.dan; }
    getKey() { return this.key; }

    equal(masu: Masu) { return ((masu.suji === this.suji) && (masu.dan === this.dan)); }

    toString() { return `${this.suji} ${this.dan}`; }
}

// 盤上にすべてのマスを含むクラス
class MasuList {
    reflist: Ref<Masu[]> = ref([]); // Masuクラスのオブジェクトのリスト

    constructor() {
        for (let s = 1; s <= 9; s++) {
            for (let d = 1; d <= 9; d++) {
                this.reflist.value.push(new Masu(s, d));
            }
        }
    }

    getList() { return this.reflist.value; }
}

// 駒クラス
class Koma {
    name: string;
    senteFlag: boolean;
    nariFlag: boolean;
    symbolid: string;
    priority: number;

    constructor(name: string, senteFlag: boolean, nariFlag: boolean) {
        this.name = name; // 駒を表わす文字列 (例: "歩")
        this.senteFlag = senteFlag; // 先手を示すFlag (false なら後手)
        this.nariFlag = nariFlag; // 成りを示すFlag
        this.symbolid = "";  // symbol の id
        this.priority = 0;  // 駒の優先順位 (数字が大きい方が優先)
        this.setSymbolID();
    }

    // this.name, this.senteFlag, this.nariFlag を元にして this.symbolid をセットする
    // 優先順位もセットする
    setSymbolID() {
        switch (this.name) {
            case "歩":
                this.priority = 10;
                if (this.senteFlag) {
                    if (this.nariFlag)
                        this.symbolid = "#S-TO";
                    else
                        this.symbolid = "#S-FU";
                } else {
                    if (this.nariFlag)
                        this.symbolid = "#G-TO";
                    else
                        this.symbolid = "#G-FU";
                }
                break;
            case "飛":
                this.priority = 92;
                if (this.senteFlag) {
                    if (this.nariFlag)
                        this.symbolid = "#S-RY";
                    else
                        this.symbolid = "#S-HI";
                } else {
                    if (this.nariFlag)
                        this.symbolid = "#G-RY";
                    else
                        this.symbolid = "#G-HI";
                }
                break;
            case "角": // 
                this.priority = 91;
                if (this.senteFlag) {
                    if (this.nariFlag)
                        this.symbolid = "#S-UM";
                    else
                        this.symbolid = "#S-KA";
                } else {
                    if (this.nariFlag)
                        this.symbolid = "#G-UM";
                    else
                        this.symbolid = "#G-KA";
                }
                break;
            case "香":
                this.priority = 71;
                if (this.senteFlag) {
                    if (this.nariFlag)
                        this.symbolid = "#S-NY";
                    else
                        this.symbolid = "#S-KY";
                } else {
                    if (this.nariFlag)
                        this.symbolid = "#G-NY";
                    else
                        this.symbolid = "#G-KY";
                }
                break;
            case "桂":
                this.priority = 72;
                if (this.senteFlag) {
                    if (this.nariFlag)
                        this.symbolid = "#S-NK";
                    else
                        this.symbolid = "#S-KE";
                } else {
                    if (this.nariFlag)
                        this.symbolid = "#G-NK";
                    else
                        this.symbolid = "#G-KE";
                }
                break;
            case "銀":
                this.priority = 81;
                if (this.senteFlag) {
                    if (this.nariFlag)
                        this.symbolid = "#S-NG";
                    else
                        this.symbolid = "#S-GI";
                } else {
                    if (this.nariFlag)
                        this.symbolid = "#G-NG";
                    else
                        this.symbolid = "#G-GI";
                }
                break;
            case "金":
                this.priority = 82;
                if (this.senteFlag) {
                    this.symbolid = "#S-KI";
                } else {
                    this.symbolid = "#G-KI";
                }
                break;
            case "王":
                this.priority = 102;
                if (this.senteFlag) {
                    this.symbolid = "#S-OU";
                } else {
                    this.symbolid = "#G-OU";
                }
                break;
            case "玉":
                this.priority = 101;
                if (this.senteFlag) {
                    this.symbolid = "#S-GY";
                } else {
                    this.symbolid = "#G-GY";
                }
                break;
        }
    }

    // name をセットする (玉と王を入れ替える操作に使う)
    setName(name: string) {
        this.name = name;
        this.setSymbolID();
    }

    // 後手から先手に変える (駒の向きが変わる)
    toSente() {
        if (!this.senteFlag) {
            this.senteFlag = true;
            this.setSymbolID();
        }
    }

    // 先手から後手に変える (駒の向きが変わる)
    toGote() {
        if (this.senteFlag) {
            this.senteFlag = false;
            this.setSymbolID();
        }
    }

    // 先後逆にする
    toSengoGyaku() {
        this.senteFlag = (!this.senteFlag);
        this.setSymbolID();
    }

    // 不成りの状態から成りの状態に変える
    toNari() {
        if ((!this.nariFlag) && this.hasUra()) {
            this.nariFlag = true;
            this.setSymbolID();
        }
    }

    // 成りの状態から不成りの状態に変える
    toFunari() {
        if (this.nariFlag) {
            this.nariFlag = false;
            this.setSymbolID();
        }
    }

    // 裏返しにする
    toUra() {
        if (this.hasUra()) {
            this.nariFlag = (!this.nariFlag);
            this.setSymbolID();
        }
    }

    // 裏側がある駒かどうか
    hasUra() {
        return ((this.name != "王") && (this.name != "玉") && (this.name != "金"));
    }

    // オブジェクトのコピーを返す
    getCopy() { return new Koma(this.name, this.senteFlag, this.nariFlag); }

    getSymbolid() { return this.symbolid; }
    getName() { return this.name; }
    getPriority() { return this.priority; }

    // Vue.js のリストレンダリングで使用する key を取得 (symbolid を使用)
    getKey() { return this.symbolid; }

    isSente() { return this.senteFlag; }
    toString() { return this.getSymbolid(); }
}

// 盤上にある駒クラス
class BanKoma {
    koma: Koma;
    masu: Masu;

    constructor(koma: Koma, masu: Masu) {
        this.koma = koma; // 駒 (Komaクラスのオブジェクト)
        this.masu = masu; // マス (Masuクラスのオブジェクト)
    }

    at(masu: Masu) { return this.masu.equal(masu); }  // BanKoma が指定のマスにあるかどうか

    // 駒を新しいマスに動かす
    moveTo(newMasu: Masu) { this.masu = newMasu; }

    // Vue.js のリストレンダリングで使用する key を取得するゲッター
    getKey() { return `${this.masu.getSuji()}${this.masu.getDan()}`; }

    isSente() { return this.koma.isSente(); }
    getSymbolid() { return this.koma.getSymbolid(); }
    getName() { return this.koma.getName(); }

    getKoma() { return this.koma.getCopy(); }  // Komaオブジェクトのコピーを返す
    getMasu() { return this.masu; }
    getSuji() { return this.masu.getSuji(); }
    getDan() { return this.masu.getDan(); }

    toString() { return `${this.masu.getSuji()}-${this.masu.getDan()}-${this.koma.toString()}`; }
}

// 盤上にあるすべての駒を含むクラス
class BanKomaList {
    reflist: Ref<BanKoma[]> = ref([]);
    preClickMasu: Masu | null;
    preClickHTMLElem: HTMLElement | null;

    constructor() {
        this.reflist = ref([]); // BanKomaクラスのオブジェクトのリスト
        this.preClickMasu = null;  // ひとつ前にクリックされたマス
        this.preClickHTMLElem = null;  // ひとつ前にクリックされた HTML element
    }

    add(koma: Koma, masu: Masu) {
        if (this.hasKomaAt(masu)) {
            // console.log("すでに駒がある場所には持駒を打てません。"); // debug
        } else {
            this.getList().push(new BanKoma(koma, masu));
        }
    }

    remove(masu: Masu) {
        const idx = this.getIndexAt(masu);
        if (idx != -1) {
            this.getList().splice(idx, 1);
        }
    }

    // 駒を動かす
    // 移動先に相手の駒があれば返す
    moveTo(nowMasu: Masu, nextMasu: Masu) {
        if (this.hasKomaAt(nowMasu)) {
            let torareKoma = null;
            if (this.hasKomaAt(nextMasu)) {
                const moveKoma = banKomaList.getKoma(nowMasu); // 移動する駒
                torareKoma = banKomaList.getKoma(nextMasu); // 移動先にある駒

                if (moveKoma && torareKoma && (moveKoma.isSente() === torareKoma.isSente())) {
                    // console.log("味方の駒がある場所には移動できません。"); // debug
                    return null;
                } else {
                    // 盤から駒を削除
                    banKomaList.remove(nextMasu);
                }
            }

            // 駒を動かす
            this.getList()[this.getIndexAt(nowMasu)].moveTo(nextMasu);

            // 取った駒を返す
            return torareKoma;
        } else {
            // console.log("元の位置に駒がありません");
            return null;
        }
    }

    getKoma(masu: Masu) {
        const idx = this.getIndexAt(masu);
        if (idx != -1) {
            return this.getList()[idx].getKoma();
        }
        return null;
    }

    // 指定したマスにある駒の index を返す (なければ -1 を返す)
    getIndexAt(masu: Masu) { return this.getList().findIndex((bk) => bk.at(masu)); }

    // 指定した位置に駒があるかどうかを返す
    hasKomaAt(masu: Masu) { return (this.getIndexAt(masu) != -1); }

    getList() { return this.reflist.value; }

    // クリックされたマスの class を変える
    setElemClass(elem: HTMLElement) {
        this.preClickHTMLElem = elem;
        this.preClickHTMLElem.classList.replace('square', 'clicked');
    }

    // マスの class を元に戻す
    resetElemClass() {
        this.preClickHTMLElem?.classList.replace('clicked', 'square');
        this.preClickHTMLElem = null;
    }

    // クリックをリセットする
    resetClick() {
        if (this.wasClicked()) {
            this.preClickMasu = null;
            this.resetElemClass();
        }
    }

    // ひとつ前にクリックされているかどうかを示す
    wasClicked() { return (this.preClickMasu != null); }

    // ひとつ前にクリックされたマスを返す
    getPreClickMasu() { return this.preClickMasu; }

    // ひとつ前にクリックされた駒を削除して返す
    pickPreClickKoma() {
        let koma = null;
        if (this.preClickMasu) {
            koma = this.getKoma(this.preClickMasu);
            this.remove(this.preClickMasu);
        }
        return koma;
    }

    // 他にどこもクリックされていない状態で、新たにクリックされた場合の処理
    newClick(event: Event, masu: Masu) {
        if (this.hasKomaAt(masu)) {
            this.preClickMasu = masu;
            this.setElemClass(event.target as HTMLElement);
        }
    }

    // 平手の配置になるように駒を追加する
    setHirate() {
        this.add(new Koma("歩", true, false), new Masu(1, 7));
        this.add(new Koma("歩", true, false), new Masu(2, 7));
        this.add(new Koma("歩", true, false), new Masu(3, 7));
        this.add(new Koma("歩", true, false), new Masu(4, 7));
        this.add(new Koma("歩", true, false), new Masu(5, 7));
        this.add(new Koma("歩", true, false), new Masu(6, 7));
        this.add(new Koma("歩", true, false), new Masu(7, 7));
        this.add(new Koma("歩", true, false), new Masu(8, 7));
        this.add(new Koma("歩", true, false), new Masu(9, 7));
        this.add(new Koma("飛", true, false), new Masu(2, 8));
        this.add(new Koma("角", true, false), new Masu(8, 8));
        this.add(new Koma("香", true, false), new Masu(1, 9));
        this.add(new Koma("桂", true, false), new Masu(2, 9));
        this.add(new Koma("銀", true, false), new Masu(3, 9));
        this.add(new Koma("金", true, false), new Masu(4, 9));
        this.add(new Koma("玉", true, false), new Masu(5, 9));
        this.add(new Koma("金", true, false), new Masu(6, 9));
        this.add(new Koma("銀", true, false), new Masu(7, 9));
        this.add(new Koma("桂", true, false), new Masu(8, 9));
        this.add(new Koma("香", true, false), new Masu(9, 9));

        this.add(new Koma("香", false, false), new Masu(1, 1));
        this.add(new Koma("桂", false, false), new Masu(2, 1));
        this.add(new Koma("銀", false, false), new Masu(3, 1));
        this.add(new Koma("金", false, false), new Masu(4, 1));
        this.add(new Koma("玉", false, false), new Masu(5, 1));
        this.add(new Koma("金", false, false), new Masu(6, 1));
        this.add(new Koma("銀", false, false), new Masu(7, 1));
        this.add(new Koma("桂", false, false), new Masu(8, 1));
        this.add(new Koma("香", false, false), new Masu(9, 1));
        this.add(new Koma("角", false, false), new Masu(2, 2));
        this.add(new Koma("飛", false, false), new Masu(8, 2));
        this.add(new Koma("歩", false, false), new Masu(1, 3));
        this.add(new Koma("歩", false, false), new Masu(2, 3));
        this.add(new Koma("歩", false, false), new Masu(3, 3));
        this.add(new Koma("歩", false, false), new Masu(4, 3));
        this.add(new Koma("歩", false, false), new Masu(5, 3));
        this.add(new Koma("歩", false, false), new Masu(6, 3));
        this.add(new Koma("歩", false, false), new Masu(7, 3));
        this.add(new Koma("歩", false, false), new Masu(8, 3));
        this.add(new Koma("歩", false, false), new Masu(9, 3));
    }
}

// 持駒クラス
class MochiKoma {
    koma: Koma;
    n: number;
    senteFlag: boolean;

    constructor(koma: Koma, n: number, senteFlag: boolean) {
        this.koma = new Koma(koma.getName(), true, false); // 持駒はすべて先手の駒と同じ向き
        this.n = n; // 個数
        this.senteFlag = senteFlag; // 先手の持駒かどうか
    }

    inc() { this.n++; }
    dec() { this.n--; }

    isSente() { return this.senteFlag; }

    getKey() { return this.koma.getKey(); }

    getSymbolid() { return this.koma.getSymbolid(); }
    getName() { return this.koma.getName(); }

    getKoma() { return this.koma.getCopy(); }
    getN() { return this.n; }

    toString() { return `${this.koma.getSymbolid()}-${this.getN()}`; }
}

// すべての持駒を含むクラス (先手、後手別)
class MochiKomaList {
    reflist: Ref<MochiKoma[]> = ref([]); // MochiKomaクラスのオブジェクトのリスト
    senteFlag: boolean; // 先手の持駒かどうか
    preClickIndex: number = -1; // ひとつ前にクリックされた持駒の index
    preClickHTMLElem: HTMLElement | null = null; // ひとつ前にクリックされた HTML element

    constructor(senteFlag: boolean) {
        this.senteFlag = senteFlag; 
    }

    add(koma: Koma) {
        const idx = this.getList().findIndex((mk) => koma.getName() === mk.getName());

        if (idx === -1) {
            this.getList().push(new MochiKoma(koma, 1, this.senteFlag));
            this.getList().sort((mk1, mk2) => (mk2.getKoma().getPriority() - mk1.getKoma().getPriority()));
        } else {
            this.getList()[idx].inc();
        }
    }

    remove(idx: number) {
        if (this.getList()[idx].getN() === 1) {
            this.getList().splice(idx, 1);
        } else {
            this.getList()[idx].dec();
        }
    }

    // 盤上に打つときに使うメンバ関数
    // 後手の持駒を打つときは、後手の向きにする
    getKoma(idx: number) {
        const koma = this.getList()[idx].getKoma();
        if (!this.senteFlag) {
            koma.toGote();
        }
        return koma;
    }

    getList() { return this.reflist.value; }

    // クリックされたマスの class を変える
    setElemClass(elem: HTMLElement) {
        this.preClickHTMLElem = elem;
        this.preClickHTMLElem.classList.replace('square', 'clicked');
    }

    // マスの class を元に戻す
    resetElemClass() {
        if (this.preClickHTMLElem) {
            this.preClickHTMLElem.classList.replace('clicked', 'square');
            this.preClickHTMLElem = null;
        }
    }

    // クリックをリセットする
    resetClick() {
        if (this.wasClicked()) {
            this.preClickIndex = -1;
            this.resetElemClass();
        }
    }

    // ひとつ前にクリックされているかどうかを示す
    wasClicked() { return (this.preClickIndex != -1); }

    // ひとつ前にクリックされた持駒を打つ (komaを削除して、返す)
    pickPreClickKoma() {
        const koma = this.getKoma(this.preClickIndex)
        this.remove(this.preClickIndex);
        return koma;
    }

    // 他にどこもクリックされていない状態で、新たにクリックされた場合の処理
    newClick(event: Event, index: number) {
        this.preClickIndex = index;
        this.setElemClass(event.target as HTMLElement);
    }
}

// 右クリックメニューのクラス
class RightClickMenu {
    display: Ref<boolean> = ref(false);
    masu: Masu | null = null;
    x: Ref<number> = ref(0);
    y: Ref<number> = ref(0);
    komaSengoGyaku: Koma | null = null; // 右クリックした駒を先後逆にした駒 (koma)
    komaSengoGyakuId: Ref<string> = ref(""); // 先後逆にした駒の symbolid
    komaUra: Koma | null = null; // 右クリックした駒を裏返した駒 (koma)
    komaUraId: Ref<string> = ref(""); // 裏側にした駒の symbolid
    komaHasUra: Ref<boolean> = ref(true); // 裏側がある駒かどうか

    constructor() {
    }

    // メニューの位置とマスと駒をセットする。
    set(targetElem: HTMLElement, masu: Masu, koma: Koma) {
        this.x.value = Number(targetElem.getAttribute("x")) + 100;
        this.y.value = Number(targetElem.getAttribute("y")) - 10;

        this.masu = masu;

        // 先後逆にした駒の symbolid をセット
        this.komaSengoGyaku = koma.getCopy();
        this.komaSengoGyaku.toSengoGyaku();
        this.komaSengoGyakuId.value = this.komaSengoGyaku.getSymbolid();

        // 裏側のある駒ならば、裏にした駒の symbolid をセット
        // 王と玉を変更できるようにする
        this.komaHasUra.value = (koma.hasUra() || (koma.getName() == "王") || (koma.getName() == "玉"));
        if (this.hasUra()) {
            this.komaUra = koma.getCopy();

            if (koma.hasUra()) {
                this.komaUra.toUra();
            } else if (koma.getName() == "王") {
                this.komaUra.setName("玉");
            } else if (koma.getName() == "玉") {
                this.komaUra.setName("王");
            }

            this.komaUraId.value = this.komaUra.getSymbolid();
        }

        this.display.value = true; // メニューを表示する
    }

    // メニューを非表示にする
    hideMenu() {
        if (this.getDisplay()) {
            this.display.value = false;
        }
    }

    getDisplay() { return this.display.value; }
    getMasu() { return this.masu; }
    getX() { return this.x.value; }
    getY() { return this.y.value; }
    hasUra() { return this.komaHasUra.value; }

    getKomaSengoGyaku() { return this.komaSengoGyaku; }
    getKomaSengoGyakuId() { return this.komaSengoGyakuId.value; }

    getKomaUra() { return this.komaUra; }
    getKomaUraId() { return this.komaUraId.value; }
}

// 各クラスのインスタンス (グローバル変数) 
const masuList = new MasuList();
const banKomaList = new BanKomaList();
const senteMochiKomaList = new MochiKomaList(true);
const goteMochiKomaList = new MochiKomaList(false);
const gomibakoKomaList = new MochiKomaList(true);
const rightClickMenu = new RightClickMenu();

// 前のクリックをリセットする
function resetClickAll() {
    banKomaList.resetClick();
    senteMochiKomaList.resetClick();
    goteMochiKomaList.resetClick();
    gomibakoKomaList.resetClick();
}

// 先手の駒台がクリックされたときの処理
function clickSenteKomadai() {
    if (banKomaList.wasClicked()) {
        // 盤から駒台に駒を追加
        const preClickKoma = banKomaList.pickPreClickKoma();
        if (preClickKoma) {
            senteMochiKomaList.add(preClickKoma);
        }
    } else if (goteMochiKomaList.wasClicked()) {
        // 後手の駒台から先手の駒台へ駒を移動
        senteMochiKomaList.add(goteMochiKomaList.pickPreClickKoma());
    } else if (gomibakoKomaList.wasClicked()) {
        // 使わない駒置き場から先手の駒台へ駒を移動
        senteMochiKomaList.add(gomibakoKomaList.pickPreClickKoma());
    }

    resetClickAll();
}

// 後手の駒台がクリックされたときの処理
function clickGoteKomadai() {
    if (banKomaList.wasClicked()) {
        // 盤から駒台に駒を追加
        const preClickKoma = banKomaList.pickPreClickKoma();
        if (preClickKoma) {
            goteMochiKomaList.add(preClickKoma);
        }
    } else if (senteMochiKomaList.wasClicked()) {
        // 先手の駒台から後手の駒台へ駒を移動
        goteMochiKomaList.add(senteMochiKomaList.pickPreClickKoma());
    } else if (gomibakoKomaList.wasClicked()) {
        // 使わない駒置き場から後手の駒台へ駒を移動
        goteMochiKomaList.add(gomibakoKomaList.pickPreClickKoma());
    }

    resetClickAll();
}

// 先手の駒台の持駒がクリックされたときの処理
function clickSenteMochiKoma(event: Event, index: number) {
    resetClickAll();
    senteMochiKomaList.newClick(event, index);
}

// 後手の駒台の持駒がクリックされたときの処理
function clickGoteMochiKoma(event: Event, index: number) {
    resetClickAll();
    goteMochiKomaList.newClick(event, index);
}

// 使わない駒置き場がクリックされたときの処理
function clickGomibako() {
    if (banKomaList.wasClicked()) {
        // 駒台に駒を追加
        const preClickKoma = banKomaList.pickPreClickKoma();
        if (preClickKoma) {
            gomibakoKomaList.add(preClickKoma);
        }
    } else if (senteMochiKomaList.wasClicked()) {
        // 先手の持駒から使わない駒置き場へ移動
        gomibakoKomaList.add(senteMochiKomaList.pickPreClickKoma());
    } else if (goteMochiKomaList.wasClicked()) {
        // 後手の持駒から使わない駒置き場へ移動
        gomibakoKomaList.add(goteMochiKomaList.pickPreClickKoma());
    }

    resetClickAll();
}

// 使わない駒置き場の駒がクリックされたときの処理
function clickGomibakoKoma(event: Event, index: number) {
    resetClickAll();
    gomibakoKomaList.newClick(event, index);
}

// マスがクリックされたときの処理
function clickMasu(event: Event, masu: Masu) {
    // ひとつ前にマスがクリックされていた場合
    if (banKomaList.wasClicked()) {
        const preClickMasu = banKomaList.getPreClickMasu();
        if (preClickMasu && (!masu.equal(preClickMasu))) { // 違うマスがクリックされた場合
            const moveKoma = banKomaList.getKoma(preClickMasu); // 移動する駒

            // 駒を移動 (取った駒が返される)
            const torareKoma = banKomaList.moveTo(preClickMasu, masu);

            if (moveKoma && torareKoma) {
                if (moveKoma.isSente()) { // 先手が駒を取った場合
                    // 先手の駒台に駒を追加
                    senteMochiKomaList.add(torareKoma);
                } else { // 後手が駒を取った場合
                    // 後手の駒台に駒を追加
                    goteMochiKomaList.add(torareKoma);
                }
            }
        }

        banKomaList.resetClick();

    } else if (senteMochiKomaList.wasClicked()) { // 先手の持駒がクリックされていた場合
        if (!banKomaList.hasKomaAt(masu)) {
            banKomaList.add(senteMochiKomaList.pickPreClickKoma(), masu);
        }
        senteMochiKomaList.resetClick();
    } else if (goteMochiKomaList.wasClicked()) { // 後手の持駒がクリックされていた場合
        if (!banKomaList.hasKomaAt(masu)) {
            banKomaList.add(goteMochiKomaList.pickPreClickKoma(), masu);
        }
        goteMochiKomaList.resetClick();
    } else if (gomibakoKomaList.wasClicked()) { // 使わない駒置き場の駒がクリックされていた場合
        if (!banKomaList.hasKomaAt(masu)) {
            banKomaList.add(gomibakoKomaList.pickPreClickKoma(), masu);
        }
        gomibakoKomaList.resetClick();
    } else {
        banKomaList.newClick(event, masu);
    }
}

// マスが右グリックされたときの処理
function rightClickMasu(event: Event, masu: Masu) {
    resetClickAll();

    // クリックされたマス目に駒があればメニューを表示する
    const koma = banKomaList.getKoma(masu);
    if (koma) {
        if (event.target) {
            rightClickMenu.set(event.target as HTMLElement, masu, koma);
        }
    }
}

// 右クリックした駒を先後逆にする
function clickSetSengoGyaku() {
    const masu = rightClickMenu.getMasu();
    const komaSengoGyaku = rightClickMenu.getKomaSengoGyaku();

    if (masu && komaSengoGyaku) {
        banKomaList.remove(masu);
        banKomaList.add(komaSengoGyaku, masu);
    }

    rightClickMenu.hideMenu();
}

// 右クリックした駒を裏返す
function clickSetUra() {
    const masu = rightClickMenu.getMasu();
    const komaUra = rightClickMenu.getKomaUra();

    if (masu && komaUra) {
        banKomaList.remove(masu);
        banKomaList.add(komaUra, masu);
    }

    rightClickMenu.hideMenu();
}

// コンポーネントがマウントされる直前に呼び出されるフックを登録
onBeforeMount(() => {
    banKomaList.setHirate();
})

</script>
                    
<template>
    <KomaSymbols />

    <svg width="100%" height="100%" viewBox="0 0 1190 1190" xmlns="http://www.w3.org/2000/svg"
        preserveAspectRatio="xMidYMin meet">

        <defs>
            <g id="stars" style="stroke: black; fill: black;">
                <polygon points="0,-6 6,0 0,6 -6,0" />
            </g>
        </defs>

        <rect x="0" y="0" width="1190" height="1190" style="fill: #8b968d;" />

        <!-- 先手の持駒 -->
        <g transform="translate(1080, 50)">
            <!-- 駒台全体 -->
            <rect width="100" height="920" x="0" y="0" style="fill: #f7c167;" />
            <rect @click="clickSenteKomadai" width="100" height="920" x="0" y="0" class="square" />

            <use href="#Sente" x="20" y="-50" width="60" height="60" />

            <g transform="translate(10, 20)">
                <!-- 持駒 -->
                <use width="80" height="80" v-for="(k, idx) in senteMochiKomaList.getList()" :key="k.getKey()" x="0"
                    :y="idx * 80" :href="k.getSymbolid()" />

                <!-- 各持駒の個数 -->
                <g id="nMochiKomaSente"
                    style="text-anchor: start; font-size: 30px; font-weight: bold; stroke: white; fill: red;">
                    <text v-for="(k, idx) in senteMochiKomaList.getList()" :key="k.getKey()" x="60" :y="idx * 80 + 70">{{
                        k.getN() > 1 ? k.getN() : '' }}</text>
                </g>

                <!-- 持駒の場所のマス -->
                <rect v-for="(k, idx) in senteMochiKomaList.getList()" @click="clickSenteMochiKoma($event, idx)" width="80"
                    height="80" class="square" :key="k.getKey()" x="0" :y="idx * 80" />
            </g>
        </g>

        <!-- 後手の持駒 -->
        <g transform="translate(10, 50)">
            <!-- 駒台全体 -->
            <rect width="100" height="920" x="0" y="0" style="fill: #f7c167;" />
            <rect @click="clickGoteKomadai" width="100" height="920" class="square" x="0" y="0" />

            <use href="#Gote" x="20" y="-50" width="60" height="60" />

            <g transform="translate(10, 20)">
                <!-- 持駒 -->
                <use width="80" height="80" v-for="(k, idx) in goteMochiKomaList.getList()" :key="k.getKey()" x="0"
                    :y="idx * 80" :href="k.getSymbolid()" />

                <!-- 各持駒の個数 -->
                <g id="nMochiKomaGote"
                    style="text-anchor: start; font-size: 30px; font-weight: bold; stroke: white; fill: red;">
                    <text v-for="(k, idx) in goteMochiKomaList.getList()" :key="k.getKey()" x="60" :y="idx * 80 + 70">{{
                        k.getN() > 1 ? k.getN() : '' }}</text>
                </g>

                <!-- 持駒の場所のマス -->
                <rect v-for="(k, idx) in goteMochiKomaList.getList()" @click="clickGoteMochiKoma($event, idx)" width="80"
                    height="80" class="square" :key="k.getKey()" x="0" :y="idx * 80" />
            </g>
        </g>

        <!-- 使わない駒置き場 -->
        <g transform="translate(130, 1000)">
            <rect width="920" height="100" x="0" y="0" style="fill: #f7c167;" />
            <rect @click="clickGomibako" width="920" height="100" class="square" x="0" y="0" />

            <!-- ゴミ箱アイコン -->
            <use href="#Gomibako" x="-30" y="0" width="60" height="60" style="fill: #666666;" />

            <g transform="translate(40, 0)">
                <!-- 駒 -->
                <use width="80" height="80" v-for="(k, idx) in gomibakoKomaList.getList()" :key="k.getKey()" :x="idx * 80"
                    y="10" :href="k.getSymbolid()" />

                <!-- 各駒の個数 -->
                <g style="text-anchor: middle; font-size: 30px; font-weight: bold; stroke: white; fill: red;">
                    <text v-for="(k, idx) in gomibakoKomaList.getList()" :key="k.getKey()" :x="idx * 80 + 40" y="100">{{
                        k.getN() > 1 ? k.getN() : '' }}</text>
                </g>

                <!-- 駒の場所のマス -->
                <rect v-for="(k, idx) in gomibakoKomaList.getList()" @click="clickGomibakoKoma($event, idx)" width="80"
                    height="80" class="square" :key="k.getKey()" :x="idx * 80" y="10" />
            </g>
        </g>


        <!-- 段の数字 -->
        <g id="numbers-row" transform="translate(190, 50)" style="text-anchor: middle; font-size: 20px; font-weight: bold;">
            <text x="0" y="0">9</text>
            <text x="100" y="0">8</text>
            <text x="200" y="0">7</text>
            <text x="300" y="0">6</text>
            <text x="400" y="0">5</text>
            <text x="500" y="0">4</text>
            <text x="600" y="0">3</text>
            <text x="700" y="0">2</text>
            <text x="800" y="0">1</text>
        </g>

        <!-- 筋の数字 -->
        <g id="numbersc-column" transform="translate(1050, 110)"
            style="text-anchor: start; font-size: 20px; font-weight: bold;" dominant-baseline="central">
            <text x="0" y="0">一</text>
            <text x="0" y="100">二</text>
            <text x="0" y="200">三</text>
            <text x="0" y="300">四</text>
            <text x="0" y="400">五</text>
            <text x="0" y="500">六</text>
            <text x="0" y="600">七</text>
            <text x="0" y="700">八</text>
            <text x="0" y="800">九</text>
        </g>

        <!-- 盤面 -->
        <g transform="translate(130, 50)">
            <rect x="0" y="0" width="920" height="920" style="fill: #f7c167;" />

            <!-- 盤面のマスと駒 -->
            <g transform="translate(10, 10)">
                <g>
                    <!-- マス目よりも駒を先に追加して、マス目が上にくるようにする -->
                    <use width="100" height="100" v-for="bk in banKomaList.getList()" :x="(9 - bk.getSuji()) * 100"
                        :y="(bk.getDan() - 1) * 100" :href="bk.getSymbolid()" :key="bk.getKey()" />
                </g>

                <g style="stroke: black; stroke-width: 2;">
                    <rect v-for="ms in masuList.getList()" @click.left="clickMasu($event, ms)"
                        @click.right.prevent="rightClickMasu($event, ms)" width="100" height="100" class="square"
                        :key="ms.getKey()" :x="(9 - ms.getSuji()) * 100" :y="(ms.getDan() - 1) * 100" />

                    <use href="#stars" x="300" y="300" />
                    <use href="#stars" x="600" y="300" />
                    <use href="#stars" x="300" y="600" />
                    <use href="#stars" x="600" y="600" />
                </g>
            </g>
        </g>

        <!-- 駒を右クリックしたときに表示されるメニュー -->
        <rect v-if="rightClickMenu.getDisplay()" @click="rightClickMenu.hideMenu()"
            @click.right.prevent="rightClickMenu.hideMenu()" x="0" y="0" width="1190" height="1190"
            style="fill: #8b968d; fill-opacity: 0.7;" />
        <g transform="translate(140, 60)">
            <g v-if="rightClickMenu.getDisplay()"
                :transform="'translate(' + rightClickMenu.getX() + ', ' + rightClickMenu.getY() + ')'">
                <!-- 裏側 -->
                <g transform="translate(0, 0)" v-if="rightClickMenu.hasUra()">
                    <rect width="120" height="120" style="fill: #666666; fill-opacity: 0.9;" />
                    <g transform="translate(10, 10)">
                        <use width="100" height="100" :href="rightClickMenu.getKomaUraId()" />
                        <rect width="100" height="100" class="square" @click="clickSetUra" />
                    </g>
                </g>

                <!-- 先後逆 -->
                <g :transform="'translate(0, ' + (rightClickMenu.hasUra() ? 120 : 0) + ')'">
                    <rect width="120" height="110" style="fill: #666666; fill-opacity: 0.9;" />
                    <g transform="translate(10, 0)">
                        <use width="100" height="100" :href="rightClickMenu.getKomaSengoGyakuId()" />
                        <rect width="100" height="100" class="square" @click="clickSetSengoGyaku" />
                    </g>
                </g>
            </g>
        </g>

    </svg>
</template>
            
<style scoped>
.clicked {
    fill-opacity: 0.4;
    fill: red;
}

.square {
    fill-opacity: 0;
    fill: white;
}

.square:hover {
    fill-opacity: 0.4;
    fill: yellow;
    transition-property: none;
}
</style>
        