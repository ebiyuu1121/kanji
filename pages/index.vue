<template>
  <v-layout column justify-center align-center>
    <v-card v-if="status=='index'" class="ma-5">
      <v-card-title>カンジア～テル</v-card-title>
      <v-card-text>このゲームは、漢字の下半分をみてどのような漢字か当てるゲームです。生成のみなので 判定は自分で行ってください。</v-card-text>
      <v-card-text>
        出現する漢字は、
        <a href="https://kanken.jitenon.jp/haitou/">漢検1～10級</a>に準拠しています。
      </v-card-text>
      <v-card-text>
        <v-btn color="primary" @click="status='setup'; boxVisible=true">プレイする！</v-btn>
      </v-card-text>
    </v-card>

    <v-btn class="ma-5" v-if="status!='index'" @click="status='index'">TOPに戻る</v-btn>

    <v-card v-if="status=='setup'" width="100%" max-width="600px" class="ma-5">
      <v-card-title>ゲーム設定</v-card-title>
      <v-card-text>
        <v-slider label="難易度" min="0" max="100" v-model="hiddenParcentage" thumb-label></v-slider>
        <v-select :items="['上','下','右','左']" v-model="hiddenDirection" label="隠す方向"></v-select>
        <v-select :items="Object.keys(kanjiDict)" v-model="activeKanji" multiple label="出現する漢字"></v-select>
        <v-btn color="primary" @click="status='game'; nextKanji()">ゲームスタート！</v-btn>
      </v-card-text>
    </v-card>

    <v-card v-if="status=='setup' || status=='game'">
      <v-card-text>
        <svg width="300" height="300">
          <text
            x="150"
            y="150"
            text-anchor="middle"
            dominant-baseline="central"
            font-size="150pt"
            stroke="black"
            stroke-width="0.3px"
            fill="black"
          >{{currentKanji}}</text>
          <rect
            v-if="boxVisible"
            :x="rectParams.x"
            :y="rectParams.y"
            :width="rectParams.width"
            :height="rectParams.height"
            fill="black"
          />
        </svg>
      </v-card-text>
      <v-card-text v-if="status=='game'">
        <v-btn color="primary" v-if="boxVisible" @click="boxVisible=false">答えを見る！</v-btn>
        <v-btn color="primary" v-else @click="nextKanji">次の問題へ！</v-btn>
      </v-card-text>
    </v-card>

    <v-card class="ma-5">
      <v-card-text>元ネタの動画はこちら↓↓↓↓</v-card-text>
      <v-card-text>
        <iframe
          width="280"
          height="155"
          src="https://www.youtube.com/embed/Vc3cIMTvZTc"
          frameborder="0"
          allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
          allowfullscreen
        ></iframe>
      </v-card-text>
    </v-card>
  </v-layout>
</template>

<script>
export default {
  data: function() {
    return {
      status: "index",
      currentKanji: "漢",
      boxVisible: true,
      hiddenDirection: "上",
      hiddenParcentage: 60,
      activeKanji: ["10級","1級"],
      kanjiDict: {
        "10級":
          "百文木本名目立力林六町天田土二日入年白八先早草足村大男竹中虫人水正生青夕石赤千川耳七車手十出女小上森五口校左三山子四糸字学気九休玉金空月犬見一右雨円王音下火花貝",
        "9級":
          "毎妹万明鳴毛門夜野友用曜来里理話肉馬売買麦半番父風分聞米歩母方北通弟店点電刀冬当東答頭同道読内南前組走多太体台地池知茶昼長鳥朝直場色食心新親図数西声星晴切雪船線矢姉思紙寺自時室社弱首秋週春書少光考行高黄合谷国黒今才細作算止市近兄形計元言原戸古午後語工公広交角楽活間丸岩顔汽記帰弓牛魚京強教引羽雲園遠何科夏家歌画回会海絵外",
        "8級":
          "由油有遊予羊洋葉陽様落流旅両緑礼列練路和氷表秒病品負部服福物平返勉放味命面問役薬湯登等動童農波配倍箱畑発反坂板皮悲美鼻筆炭短談着注柱丁帳調追定庭笛鉄転都度投豆島真深進世整昔全相送想息速族他打対待代第題拾終習集住重宿所暑助昭消商章勝乗植申身神仕死使始指歯詩次事持式実写者主守取酒受州銀区苦具君係軽血決研県庫湖向幸港号根祭皿開階寒感漢館岸起期客究急級宮球去橋業曲局悪安暗医委意育員院飲運泳駅央横屋温化荷界",
        "7級":
          "約勇要養浴利陸良料量輪類令冷例歴連老労録夫付府副粉兵別辺変便包法望牧末満未脈民無徒努灯堂働特得毒熱念敗梅博飯飛費必票標不束側続卒孫帯隊達単置仲貯兆腸低底停的典伝照賞臣信成省清静席積折節説浅戦選然争倉巣士氏史司試児治辞失借種周祝順初松笑唱焼象固功好候航康告差菜最材昨札刷殺察参産散残給挙漁共協鏡競極訓軍郡径型景芸欠結建健験街各覚完官管関観願希季紀喜旗器機議求泣救愛案以衣位囲胃印英栄塩億加果貨課芽改械害",
        "6級":
          "豊防貿暴務夢迷綿輸余預容略留領非備俵評貧布婦富武復複仏編弁保墓報程適敵統銅導徳独任燃能破犯判版比肥総造像増則測属率損退貸態団断築張提職制性政勢精製税責績接設舌絶銭祖素質舎謝授修述術準序招承証条状常情織採際在財罪雑酸賛支志枝師資飼示似識現減故個護効厚耕鉱構興講混査再災妻逆久旧居許境均禁句群経潔件券険検限河過賀快解格確額刊幹慣眼基寄規技義圧移因永営衛易益液演応往桜恩可仮価",
        "5級":
          "幼欲翌乱卵覧裏律臨朗論閉片補暮宝訪亡忘棒枚幕密盟模訳郵優認納脳派拝背肺俳班晩否批秘腹奮並陛暖値宙忠著庁頂潮賃痛展討党糖届難乳染善奏窓創装層操蔵臓存尊宅担探誕段将傷障城蒸針仁垂推寸盛聖誠宣専泉洗捨尺若樹収宗就衆従縦縮熟純処署諸除骨困砂座済裁策冊蚕至私姿視詞誌磁射穴絹権憲源厳己呼誤后孝皇紅降鋼刻穀簡危机揮貴疑吸供胸郷勤筋系敬警劇激異遺域宇映延沿我灰拡革閣割株干巻看",
        "4級":
          "丈畳殖飾触侵振浸寝慎震薪尽陣 秀襲柔獣瞬旬巡盾召床沼称紹詳脂紫雌執芝斜煮釈寂朱狩趣需舟稿豪込婚鎖彩歳載剤咲惨旨伺刺堅遣玄枯誇鼓互抗攻更恒荒香項駆屈掘繰恵傾継迎撃肩兼剣軒圏朽巨拠距御凶叫狂況狭恐響驚仰鑑含奇祈鬼幾輝儀戯詰却脚及丘介戒皆壊較獲刈甘汗乾勧歓監環鋭越援煙鉛縁汚押奥憶菓暇箇雅握扱依威為偉違維緯壱芋陰隠影暦劣烈恋露郎惑腕 翼雷頼絡欄離粒慮療隣涙隷齢麗娘茂猛網黙紋躍雄与誉溶腰踊謡忙坊肪冒傍帽凡盆慢漫妙眠矛霧敷膚賦舞幅払噴柄壁捕舗抱峰砲彼疲被避尾微匹描浜敏怖浮普腐拍泊迫薄爆髪抜罰般販搬範繁盤塔稲踏闘胴峠突鈍曇弐悩濃杯輩添殿吐途渡奴怒到逃倒唐桃透盗恥致遅蓄沖跳徴澄沈珍抵堤摘滴贈即俗耐替沢拓濁脱丹淡嘆端弾尋吹是井姓征跡占扇鮮訴僧燥騒",
        "3級":
          "嘱辱伸辛審炊粋衰酔遂穂随髄 如徐匠昇掌晶焦衝鐘冗嬢錠譲施諮侍慈軸疾湿赦邪殊寿潤遵恨紺魂墾債催削搾錯撮擦暫祉巧甲坑拘郊控慌硬絞綱酵克獄憩鶏鯨倹賢幻孤弧雇顧娯悟孔峡脅凝斤緊愚偶遇刑契啓掲携忌軌既棋棄騎欺犠菊吉喫虐虚岳掛滑肝冠勘貫喚換敢緩企岐架華嫁餓怪悔塊慨該概郭隔穫哀慰詠悦閲炎宴欧殴乙卸穏佳励零霊裂廉錬炉浪廊楼漏湾揚揺擁抑裸濫吏隆了猟陵糧厘墨没翻魔埋膜又魅滅免幽誘憂邦奉胞倣崩飽縫乏妨房某膨謀苗赴符封伏覆紛墳癖募慕簿芳陪縛伐帆伴畔藩蛮卑碑泌姫漂哲斗塗凍陶痘匿篤豚尿粘婆排窒抽鋳駐彫超聴陳鎮墜帝訂締逮滞滝択卓託諾奪胆鍛壇稚畜粗礎双桑掃葬遭憎促賊怠胎袋瀬牲婿請斥隻惜籍摂潜繕阻措",
        準2級:
          "彰償礁浄剰縄壌醸津唇娠紳診刃迅 緒叙升抄肖尚宵症祥渉訟硝粧詔奨酬醜汁充渋銃叔淑粛塾俊准殉循庶肢嗣賜滋璽漆遮蛇酌爵珠儒囚臭愁酷昆懇佐唆詐砕宰栽斎崎索酢桟傘顕懸弦呉碁江肯侯洪貢溝衡購拷剛謹襟吟隅勲薫茎渓蛍慶傑嫌献謙繭頑飢宜偽擬糾窮拒享挟恭矯暁菌琴渇褐轄且缶陥患堪棺款閑寛憾還艦靴寡稼蚊拐懐劾涯垣核殻嚇潟括喝亜尉逸姻韻畝浦疫謁猿凹翁虞渦禍鈴賄枠 羅酪痢履柳竜硫虜涼僚寮倫累塁戻銘妄盲耗厄愉諭癒唯悠猶裕融庸窯泡俸褒剖紡朴僕撲堀奔麻摩磨抹岬瓶扶附譜侮沸雰憤丙併塀幣弊偏遍伯舶漠肌鉢閥煩頒妃披扉罷猫賓頻洞督凸屯軟尼妊忍寧把覇廃培媒賠亭貞逓偵艇泥迭徹撤悼搭棟筒謄騰嫡衷弔挑眺釣懲勅朕塚漬坪呈廷邸喪槽霜藻妥堕惰駄泰濯但棚痴逐秩旋践遷薦繊禅漸租疎塑壮荘捜挿曹甚帥睡枢崇据杉斉逝誓析拙窃仙栓",
        "2級":
          "挨曖宛嵐畏萎椅彙茨咽淫唄鬱怨媛艶旺岡臆俺苛牙瓦楷潰諧崖蓋骸柿顎葛釜鎌韓玩伎亀毀畿臼嗅巾僅錦惧串窟熊詣憬稽隙桁拳鍵舷股虎錮采塞埼柵刹拶斬恣摯勾梗喉乞傲駒頃痕沙挫餌尻芯腎須裾凄醒脊戚鹿叱嫉腫呪袖羞蹴憧拭煎羨腺詮箋膳狙遡曽爽痩踪捉遜汰唾堆戴誰旦綻緻酎貼嘲捗椎爪鶴諦溺填妬賭藤瞳栃頓貪丼那奈梨謎鍋匂虹捻罵剥箸氾汎阪斑眉膝肘阜訃蔽餅璧蔑哺蜂貌頬睦勃昧枕蜜冥麺冶弥闇喩湧妖呂賂弄籠麓脇瘍沃拉辣藍璃慄侶瞭瑠",
        準1級:
          "娃唖蛙窪阿啞唖啞娃逢饗襖碧蒼葵葱碧蒼梧煽煽簸垢緋赫嬰莱萊茜昂龝穐叡彬亮灼燦晃厭倦呆渥啞偓唖鹸鹼芥倦緋曙旭詑燦葦疋鰺鯵翫梓畦妓娼嬉恰仇斡敦醇竺渥湛淳惇鍾鳩湊揖鳩輯蒐鍾纂蹟瀆翫涜姐姐肋溢蝱虻鐙膏蛋甜飴彪綾斐絢殆賎賤謬鮎疏蕩砿礦匪灼砺礪蘭蟻蛾或或莱萊蕪粟沫袷憐憐怜按鞍杏庵杏伊蔚倭鮪謂葦飴惟夷莞亥猪謂云謂愈庵筏凧碇錨熔鎔歪郁粥戎牢屑些諫諌胡烏堰磯顚顛迄溢寵伍絃厭庄埜坐鰍狗禾猪祷禱荊楚荆歪坐藷薯厭賎埜賤賤賎愈愈舛熔鎔凾函磐巌巌鰯謂寅允鸚胤蔭吋胡迂烏佑卯鵜蒔窺覗諜穿鶯鴬饗蕩蕩兎莵兔菟丑汐艮碓稀紗巴噓嘘裡苑蔚槌蟬佼繡娃繍蝉摸梁靭靱釧疏疏趨鰻畦姥甜廐廏厰廠厩膿膿倦娩楳欽裡卦卜卜卦瓜閏渥濡膏斌邑嬉鱗噂云廻慧荏瑛叡頴穎盈嬰洩曳亦榎蝦蛋夷蕃胡戎夷撰衿燕焰苑奄渕厭焔淵堰鳶蜎掩鴛薗烏於牡阿苧笈甥於鴦鸚鷹鶯鷗襖鴬鴨邑姶凰鴎厭駈樗輿套葢蔀浩祁蔭葢奄掩蒙狼宏峻恢碩鴻浩魁戎甫祁逢渠櫓紘凰鵬鳳鴻蔭荻托訣渕淵桶於煽喬浩溢綜酋按厭啞唖馴鴛鴦怯牡托捺兇煽侠俠斧栗怯劫恕惟惟阿趨錘惟爺迄澱梼魯檮呆疏坐畢竣厭薗苑禾瓜樺嘉伽嘩卦蝦茄珂榎霞迦也哉耶馨乎亥鮭罫鎧蟹芥恢魁堺咳凱漑桧苅檜晦蠏廻浬楓蛙馨馨捧繋斯繫偓鞠赫耀蛎堵蠣黛帖鉤鈎摑撹廓塙劃攪掴赫此搔掻斯郁馨駈鈎鉤蔭寓篭笠嵩葢沓套嵩扮橿樫慧叡姦酋魁頁渠柏梢舵柁梶鰍嚙噛粕糟霞霞劫掠掠掠塙犀牢仇禰祢頗僻凱恰筈捷鰹嘗桂稜崕哉乎叶鼎鎚蠏蟹鉦庚樺椛掩庇屍鞄蕪兜鏑蕪鏑竈竃噌喧嘩嚙噛禿鴨醱醗鴎鷗茅萱粥腔芥烏腔躯軀苧雁鴈蒐佃僑寓貰苅薙駈脆伊渠翆翠蒲姦菅莞侃鹼斡諌凾舘㵎灌澗桓函翰鹸竿潅諫柑萱按篦箆馨臥峨蝦蛾峩駕俄伽咳凱漑苅崕葢碍亥鎧鰐鍔巌崕蒲蝦雁癌砧鴈翫巌贋麒嬉祁毅槻其箕窺磯鰭葵稀徽葱掬麴麹鞠訊樵樵萌萠繫繋朔鉾鋒橘桔吃迄狐砧杵茸黍栗峻卿伽俠侠鳩笈玖汲廏仇韮灸厩韭廐噓鋸渠嘘饗杏亨彊袷劫兇僑蕎侠喬叶卿匡橿怯俠馨旭燦桐錐檮梼麒麟粁ー禽欽芹欣檎衿蛾誼蟻妓祇禦尭倶鈎俱軀駈鉤躯矩鳩玖狗垢杭腔喰吃餐釘劃鵠閤卿叢枇櫛櫛釧摺樟楠樟屑糞吻髭吻沓顛顚燕轡邑祇橡頸頚窪窪隈阿伍纂綬汲粂蜘蛛鞍昏晦蒙喰晦昏栗廚厨庖輿廓莫昏蟻烏鍬馴弘寓倶俱寓芥卦稀袈笥圭荆繫頸卿桂頚慧繋馨畦珪鮭荊罫涜瀆糞垢戟葢頁桔訣蕨嵯峻峨嵳巌峩倦㵎牽澗捲蜎鹸絢硯萱喧鹼鰹碍戟噛嚙諺硯倦絃彦鈷跨乎袴菰糊瑚姑壺醐胡狐壷妓仔鯉渥塙縞庚紘幌糠鉤鮫閤攪杭膏晃肴葢鵠怯宏皋叩凰皐浩礦腔釦藁亙狗垢肱昂鈎鴻巷亨砿恰撹倖袷佼弘蛤亘劫壕麴麹柑蒙梱膏冴鵠漕苔此嘗輿甑漉梶梢忽惚儘悉畢諺竪豎之這斯此溢狛狛仔稗叉叉篭菰竪豎篭梱樵斯此之惟伊這劉夷艮昏坤梱冴伍檎瑚梧醐胡吾轟昂嚙劫壕噛濠欣艮裟嵳瑳些乍蓑紗嵯叉栖哉蓑柴晒砦犀偲啐禄倖禎冴竿堺榊伶盃鍾鮭肴溯燕昌郁赫祁尖魁鷺朔窄摸諜麹鮭麴笹碍捧漣矩匙枇叉薩撒皋皐怜慧叡智聡藷諒鯖捌捌錆淋莫錆碍遥鮫鞘杷掠曝曝晒曝薮藪皋皐擾讚珊撰撒讃燦餐蒜纂坐之錫孜弛屍笥只砥梓髭斯飴覗巳祇偲此仔蒔匙獅彊汐鹸鹼而屍而而而梱鴫鋪播蔭蒼茸蕃蕪苒厰廠獅宍醬醤賎賤雫湛漉淋溜櫛悉蛭蔀靭靱篠凌駕篠偲柴屡屢姑鮪沫蕊蕋蘂凋嶋縞洲諜痔楚柘紗叉姐這灼杓勺蹟錫杓喋蛛趨諏撞咒穐鰍洲柊揖鷲繡楢嵩繍咒鍬酋輯蒐葺萩龝姑姑夙粥啐竣遁馴舜駿隼醇峻鼠薯疏岨曙恕疋渚杵黍鋤藷埜哨丞蕉鯖菖捷厰秤鍾廠甥鉦妾樟梢蛸昌醬摺蒋鍬鞘鎗蔣庄橡娼嘗篠湘錆樵醤裳菖粟埴蝕燭按遁徽瑞鵠縞咳咳訊矧疹槙辰秦賑榛晋而痔爾蒔馳竺宍柚爺惹雀惹咒濡竪嬬鷲綬豎紐什廿戎遁馴淳楯惇隼醇閏恕鋤汝薯茸牒瀞嘗鄭擾帖丞穣杖允稔儘靭荏塵訊靱壬諏雛鼠蘇笥簾栖洲棲錐瑞錘翆翠雛趨嵩吃菅鍬銚鋤箆篦椙溢鋤漉掬匡穎尤駿頴弼輔亮菅些頗煤漑煤晋苒錫鐸雀硯箔簾淳迺廼乃惣綜窄隈棲栖栖棲李摺犀尖楚坐駿吋逗杜廚厨弗蕋蘂瑞蕊銑靖脆犀錆棲鯖貰甥栖鉦瀞蹟碩錫汐堰咳堰咳屑鱈洩窄窄逼瀕蝉蟬芹茜閃蟬栴賤穿釧煽揃亘蝉苫舛箭銑賎尖撰糎ー脆蝉賎苒蟬賤楚姐疋鼠蘇溯噌疏岨蛸漕湊槍鞘鎗匝葱庄叢綜竃竈噌甑蚤帀惣聡糟鰺薮搔藪掻蒼宋鯵瀕儲燭粟趨誹漑潅潑灌溌淋坐啐苑薗爾其岨岨舛叛薙剃其揃揃揃巽噂樽鱒橡粟鱒陀騨驒詫詑舵柁黛碓岱呆腿梯鎚苔殆鯛坦夷夷仔顚顛鷹昂峻尭喬嵩鐙昂佃琢擢托鐸啄焚儲茸毅凧蛸佑祐弼毗讚祐弼輔丞讃佑毘訊湛讃讚歎叩摺惟伊只董匡舘橘掩乍奄忽韃辰竪豎巽轡舘竪楯豎辿㵎澗㵎澗狸胤嬉托餐珪圭錫蛋錫廟瀦溜潴萌萠溜溜弛訊鱈樽弛帖騨驒湛歎耽檀椴坦簞灘蛋箪橢雫楕舵柁騨驒陀迺廼乃醍悌韃捺檀椴灘馳弛蜘薙智茅智茅柚筑禿巷蛛厨紬丑蛛註紐廚儲苧猪潴瀦樗暢釘帖喋吊凋蝶脹傭挺寵鯛銚諜牒蔦肇塵箕砧椿湛疹槌鎚朔啄杖埠斡摑掴倦栂槻筑撞悉佃葺纂柘晦辻蔦鎚坤槌糞戊欽寅塘蟬蝉綴夙孜彊繫繋婁繋婁繫鍔椿燕螺悉壺壷窄嬬錘稔紬紬蚤侃彊毅禦聨聯洛牽穿劉聨聯鈎鉤絃蔓吊吊橡仇戎鄭蹄碇梯挺鼎鵜綴醍禎薙悌釘汀剃鯉翰鏑擢荻綴畷轍姪蛭顚澱佃顛纒纏甜淀辿槙祢禰胡撚淀澱鮎佃兔鍍堵兜菟兎杜莵砥樋砥禱兜喋董萄濤撞瀆沓宕套祷塘涛梼檮樋桶鐙桐蕩嶋逗淘涜吋訊諏什迂遥緬疏暢亨疏熔鎔栂尤尖辰龝穐鴇註伽啄涜竺禿瀆砺礪砥熔鎔庚稔杜綴橡椴濡椴逗轟鳶翰惚苫朋巴燭倶俱寅繫禽繋酉禽鞠砦瀞蕩吞沌呑遁敦惇噸鳶囊嚢桐膿萄撞鰍吃弗吞呑廼乃迺莫勿朋莫勿遥胡乍凪渚汀牟凪薙歎勿駁茄茄灘宥捺撫銚鮎濤涛鞄啐嘗楢擾馴也馴畷楠汝廼乃而爾迺烏胡渠爾賑賑宍螺繍繡廿贋螺韭韮烹亨薗俄俄荏壬稔蒜繡繍糠挺擢挺擢濡禰祢姐葱鼠煉鮎稔撚鄭乃埜篦迺箆之廼膿囊嚢遁鴇禾鋸鋸輿窺覗浬洩暢箔暢幡蚤餐吞呑糊矩駕駕咒簸琶頗杷播巴芭稗盃牌吠這蝿蠅捌袴秤猷猷諏萩駁柏箔狛粕穿噓嘘矧捌禿禿箪凾函笥簞漕硲梁疹梯榛趨垢孟肇甫肇蓮芙茄筈馳畠幡鰭秦叩圃畠捌椀蓮醱溌醗筏捌潑鳩埠噺叛塙埴潑溌蔓蛤夙捷蚤趨灘隼蕩糞鉤榛鈎梁遼遥緬脹叛磐幡蕃扮梓芭狽煤楳吠栂莫曝摸駁筏姥蕃播娩鰻蔓挽磐緋庇匪轡誹毘枇斐毗樋桧檜辰柊稗叩晃僻疋曳惹汲牽挽髭彦粥瓢簞箪庇阿菱醤醬鴻杓肱歪摺疋畢弼逼坤蹄牢雛撚桧檜蛾蛾僻綬紐柏彪豹彪瓢杓逼雛酉閃蒜蛭簸幡怯鰭廓紘宏恢浩弘廓弘斌牝瀕彬毘枇梶毗琵柏謬鋲錨廟秤芙甫斧撫鋪峯蒲埠鮒輔楓蕃淵湛渕蕗葺噓嘘瓢脹脹囊嚢茸鮭湛耽沌奄苑杜柴臥蘭臥襖禦葢牌竿諜牒篇禄渕粟淵弗翰莞鮒篇翰郁喋嬰糞扮焚吻鵡蔀蒲葡撫蕪鳳駁勿箆篦僻僻碧竃竈倖閤篦箆鞭篇捌瞥緬娩鞭麪頁ー鋪圃葡輔甫菩蒲鳳鵬逢峯萌亨朋捧庖蓬萠鴇烹鞄鋒蔀呆惚吠卜曝戟鋒鉾鋒詫頴穎蕩宕穿穿慾弗殆殆焔焰讚讃宕螺濠壕惚幌幡叛姥莫菩摸牡戊牟萌牡萠茅卯鉾呆戊蝱孟虻蒙穆卜惚釦哩蕃鈎鉤迂槙篇莫撒播蒔捲捲鮪諒允惇允蒋蔣菰柾柾鼎愈坐綜咒咒駁鱒攪撹亦跨叉也俣跨跨夙彪駁沫纒禱纏祷纏纒迄鵠纒纏纒纏纒纏惹稀疏儘黛蛾黛檀鞠稀麿廻廻鰻蔓孟幡箕巳柑礪琢砥砺瑳翰渚瀕汀瑞潴瀦鮫壬鋪晦涜渠瀆祢廟禰撹攪姦蛙胡蕪狼擾猷溢盈翆翠碧嬰湊嶺峯蓑穣稔哨苓姶洛邑粍ー瞥鉾蕪牟鵡溯逢麪椋舜軀躯葎翫牟沓蝕蒲苫莞掬鞭荊韃荆楚掠鞭韃廓邑叢庄牝匁姪妾捲寵萠寵萌廻匝廻帀斡嬰牝鍍瑞萌萠麪棉緬摸姥莫裳孟蝱虻蒙儲儲萌萠焰焔穆杢裳悶勿尤醇翫饗粟籾椛腿袴萌萠貰杜洩脆悶匁耶埜爺也乎箭也哉灼灸舘喧亦焚灼櫓鞠廟鏑賎靖賤靖靖鳩傭傭寓僑梁楊筈薮藪柘倭杜槍鑓鎗苒荏脆凱輯穆愈柚惟鮪楢涌厭宥柚邑酉祐猷佑揖尤歪靱靭之柚揖巽穣忽貰允宥恕弛於輿馴徽嘉誼遥蓉鎔傭頁銚蠅耀楊蝿涌鷹厭熔慾姦垢葦誼鯖寓扮淀澱淀澱淘蘇嘉蓬撚阿撚函凾鎧鎧欣嬬脆螺萊莱洛蘭栗哩李浬鯉裡狸葎栗掠溜笠琉劉遼伶梁嶺椋亮諒菱苓掠綾稜凌麟淋綾琳鱗燐淋篭屡婁琉屢玲砺苓蠣礪怜伶嶺蛎漣蓮怜聨聯簾煉憐魯鷺櫓蕗摺蝋蠟稜聾篭婁狼牢肋禄漉窪倭蛙隈歪劃訣吾亦或涌伶妓鷲萱擾棉襖亘杭亙磐轍蹄鰐詫詫藁啞唖蕨妾箪簞兇匪吾盌碗椀乎",
        "1級":
          "椏丫鐚哇堊婀猗痾閼錏鴉熙欸嗚噫咨熈欹粤嗟于猗煕噯吁齎噯隘矮靄阨埃噫穢靉鞋欸藹哇遘逅盍逑覯邂覿覲晤喘韲齏韲齏滄黽黝蜀蔬呷赭赬絳膩殷赭絳猩蚶胝腁皸皹胼孩藜赭赧皎贖貲贖亢昜驤旻慊歉慊賈估賈晣瀏曄粲晰耿曠韶晟焜晤杲瑩煌烱犖炯闡煥晢炳皖奐晳倬昿晧暘冏渙覈酲飫惘腮顋鰓軛齷噁扼搤幄堊蔡翊趺踑跏丱丫掀驤翹扛抖榕衵袙頷頤腮顋鬚晁苴蕣紵暾蜊鯏黶痣糺薊詭紿謾誣誑罔詒瞞粲爛哂嗤芦蘆葭趾趺蹤蹠跫鐐桎鐺騅晨蹇蹠跖葭笳簣咀荅堋褪悁畛阡陌倡遨游敖咫賈估歛捐燠煦煖暄煖煦嫗牴亢冤讎讐寇寃冦寇冦遖閼遏軋厖諄腆亶燠靦攢搶攅簇滔聚荐輳萃翕蔟萃翕斂纉攢攅逑纘緝聚羹誂誂椹諷嬋窕窈址趾迹墟蹤阯迹裔坎埳竇竅穹塹嵌壙窩蹠窖竇蚩褻狎謾豈嫂訐詆骭鶩渝衍膩腴膩薹蜚煬炙燔焙熹炮蜑醴苻繖醴霤泛浹溥旁洽潦仂酳畸衍贏饒罘罔汕罨罟罠辮餃黼綵綺賁陲綵詭譎馭愆瑕愆跌紕騫譌訛愆繆悍糲笨鬆桀蔬莽悍麁麤鷙盪灑澣澡浣滌洒蘯沐駻樸諍畬畭璞釐悛竄鷙霰旌垤儻穢粱澹醂褶覯戮歙幷并勠窘躁遽遑矍悤遽遑怱鮑鰒蚫愍憫恤閔矜卹黯殷鮟罨餡晏菴諳猗懿痿貽饐倚蝟矣帷噫縊已欹洟怡佗頤肄詒鰄韋彜熨彝幃恚渭姨苡痍豕藺粲咢吩咐曰廈厦廬雖鴿痊瘉菴廬霆桴槎枋柎怎儼瞋嗔鵤悁馮嗔愾恚怫瞋慍忿鵤毬范啀閾縡澳鬻燠戈艨艟藺弋溏汪偈奕勣諍聊矍諍躄躃磑礫磴甃磴鐓磴矼箴碣瓷砠弩鶍鉅焉曷孰礒匆倥怱悤檻鈑椒巓鼬黯悽恫軫惻疼慇怛悵惆悽愴癆俑撓炒臻踵訖曁雍聿弌檪櫟莓苺炳紵弌鎰鴥鴪聿佚噎鷸軼曷憮譎譖佯譎譌詭訛譛繆綫緡縷綸縷遑撩鯔蝗螽霆嘶戌蘢筥秉豕誄棘篳笆溲鼾訝訝肚熏燻疣贅肬箴誡誡箴飭撕諱諱蜀鄙隘俚仄陋苟鄙瘉痊兪逾俞甍蕁圦糴炒熬黥淹涵綺堊鑪嵒岫嵒曰鰮鰛鮇窩尹殷韵贇婬殞螾霪湮慇茵恁隕酳蚓堙喑吁嫗紆竽于盂齲桙嗚燠傴侑禹茴筌殍殍秧饑饉餒罟闖俔遉覘弋泛瀲泛漱嗽鑚鑽鐫鑿萍蘋苹鰾鯏鯎朮膺丕禀稟盪撼朶蘯蘯躁盪蠕滔蹌蠕蠢罘驢隕佚蛆磑碾澹漓澆偸菲絅繻濛舂疼疼踞蹲潘熄堙堙鶉軼鷽嘯咏謌謳哥嘔謳哥咏謌哦猜弍茹褂襠袿裲袿晤絛訌譛愬諍譖熨拊戞戛擽擂挌捶抃誅鷙搏搗擣膺鏗奕娟鑠夭妍猗姚曼倩媚娥綺韶姸婉綉鈔徙鈔搨俯靫鱓俯徙蒂蔕趺萼闍柎臂膊倆濶闊髫魘頷唸呻唸壟隴畴疇姆媼篡褫虔簒鹵蔗圉站櫪溟瀛惓孶孳呻筮筮恚悵惆慍懊鞅怏糶衒估糶賈沽售沾霑洽沐浹沾霑洽浹涵粳懿恙閔煢卹懆奕悒愍憫悄恤褞袍釉囈譫蠎蟒椁槨暈耘吽慍紜縕薀饂褞繧韞蘊彗穢秉枋柯霙郢泄咏纓裔梲兌瀛贏瓔殪癭楹瑩蠑塋翳鱏腋懌蜴掖繹奕鯣站靨刳剔刔抉鱛朶柯徭兌鉞噎戉曰饐粤噦莠蛯鰕狄蜑羯羌貊箙瘟癘厲鰓腮顋揀刪删柬銓掏魞袵衽滬柬雕鏨鏤鐫掾爰閹圜湮覃烟莚嫣焉淹罨鼴衍黶涎娟轅臙閻冤豌偃讌筵湲嚥鋺魘悁閼簷寃蝘蜒蜿婉捐檐槐嗚飫唹淤噁耄逋耋耆膺瓮謳懊鏖泓枉鞅澳閘燠秧拗鶲甕怏嘔媼罌汪殃嚶嫗甌躡趁襁朸疸楝欒杞媼嫗霈殷黎夥饒鉅臻稠藹罨冪扞羃屏屛幎盍溟鼇鼇鉅冢衍尨嵬莽奐瑰麁麤奕倬丕侈傀厖穹汪溥蘢瑟逵麋麈鼇諚穹熕逵淌茘薤蜃舸鮱弩隴墟堽坏邱奸燠澳熾謖掟諚叟嬖蒹裨苴擱跋噯噫邃窈眇袵衽窈窕謚諡饋賻贐餽餞詒餉饋賻貽朮螻熾鰧懶嬾謾懈慵疥瘧翕儼驕侈奢夸伉踞僭敖倨靡偃蹇僣尹佰筬耆熨勒扼丱孺幺聿勒撩撥馭脩艾甸臧綸釐尹斂乂誨誨魴斟揣忖吝慳悋穡嗇舅麌擠獺晏輓竦懾兢惶恟悸遽懼惴聳恂瞿悚怕歙愬懍慴魘蛞藹粤殞隕膃頤頷穽擠貶貶殞隕黜囮孱癆敝恫諢詼縅縅恫踴踉踴棘駭慫怛駭愕顫咢懼禺儺鍠釿姨牴羝夥佩珮恊剽佩湎朧朦輊侖俤羈諂佞韵趁漿拇擘游泅曁趙覃渣滓淤檻蠢聒倥惷僮矇佝駑蚩鹵笨颪鹵蟒蠎杪竟訖歿鰮薀褞鰛韞慍瘟縕遐谺跏賈萵裹舸痂堝厦哥窩譌夥戈訛謌蝸夸枷笳軻鰕顆珈葭廈瑕訶罅柯找蝌踝囮譁呵歟丐迴挂槐垓邂疥愒枴匯瑰廨薤徊醢獪駭鞋价嵬隗噦喙誡揩膾褂誨詼茴愾懈偕鱠蛔夬乖傀孩橈艪蠡楫櫂胛櫪褓糴鰄圉賈沽捐糴豢孵孚卻槭盻眷眄兌爰渝孵臉芬馥馥芬嚊嬶嚊嬶榜搴掀撥騫跟踵踝攣恁憑罹攣鑒魴跼傴僂燿煌煥燿暉曄焜燿篝燎炬縢屏屛牆墻牋拌牘鑰鎰剳垠疆疆愨椁硅覈桷矍骼槨埆狢鷽恪霍幗擱壑挌梏瓠貉癨钁蠖攫膈馘喀闕虧舁抓爬贓臧韜窩窩遯蟄竄椒筧鵥筧囓挂騫闕虧齧翔翳翳翳埒喞藉罩筺轎篝樊籃筐疔痒瘡暈疽鵲奕仍荐褶痂翳簪翳錺賁紕縟鏤紕賁爨晢俐晣傅淅顱仟槲楫榜櫂橈舳鮖忰悴齧囓咬渣滓藉緲渺暝眇髣窈鎹翳剿勦鈔綛纃絣籌鯑鬘桛枷綛飆范鞏矼膠艱狷旁冦寇尸忝頌蝸裼袒蘋帷筐筺睨仄昃陂欹鞏辟跛騙仄旁馮鞨歇劼猾瞎羯聒刮戞戛拮蛞黠濶闊豁頡蝎蠍愒曷戡搗龕贏剋扛舁鬘餉搗糅觚楞楞恊悽愴鉗鋺鐶錚銕帑麑尸黴黴菁冑鱟菁菁癆鑵剳扎搆冓叺魳框爨敖呶讙嘖譁囂聒諠啅嗷楮珈幗裃霹咀咬咥齧齦齟噬囓嚼狠瓷甕罌缸瓮甌鳬鳧羚髢茆榧鬻餬糜癢痒孅鹹旄揶捓縢紮枷鴉槁蚌鱲枳枸棠涸隍搦搦畋甸蕕偸苟殯殯藉櫚艾芟蒭乂芻畋甸佻嫖佻鰈鰔餉糒餉槁嗄涸沍竭歇槁獺槁旱槁晞鞨鞜裘躱翡鷸靺厠溷廁杞磚磧甓甎駱薛渝悍浣橄豢鼾驩扞龕皖鉗鹹瀚檻寰羹銜鰔癇湲喊鬟邯綸艱栞矜鸛鑒懽撼戡柬顴餡駻慳讙宦咸歛桿鐶箝杆啣坩頷揀稈緘嵌旱瞰鑵丱盥澣坎嫻奸圜嫺煥旰埳疳骭涵捍轗罕渙奐拑酣鰥蚶燗覈鑒笄鈿筓釵簪橇鉋巫覡閂菲馥冕訝衙鵝娥莪哦囮鵞睚剴駭豈嵬鮠啀愾皚礙磑孩艾垓乂鷽諤壑鄂鶚齶咢愕萼鵞鵝蟆蟇蒻呏碪啣嵒頷莟偐龕銜鮨饋禧沂騏逵愾卉欹跪覬熙櫃咥譏屓羇畸熹喟几奎熈冀虧朞饑掎麾綺萁匱餽簣耆晞棊綦詭暉釐煕枳圻僖亟屭祺悸諱驥羈倚欷唏剞踑豈跂癸曦揆燬杞馗曁愧氛柝銷熄鞫椈聆鶎蠹蝎蠧蕘釁剞剋鏤勒雕垠圻軋轢軋樸雉鱚痍瘡瑕釁呰疵瘢瘢痍痍縻絆鞅紲緤絏穢臻鋩拮屹頡佶訖譎髻莪帛繖碪蕈椏蘗檗稷粢剋峭覈踵辟褊剋夬卻蹻摎烋歙繆赳邱柩鬮岌咎毬躬疚蚯糺裘逑舅翕樛穹醵倨苣炬秬鉅筥吁遽歔踞墟欅疆竟嬌夾誑鞏蹻竅陜薑繈框轎孒羌筐兢磽跫囂洶僵橇莢蛬皎鋏冏姜繳筺驍梟筴驕恊繦恟歙徼拱襁匈蛩嚮篋慊矜檋棘勖洫跼亟蕀勗髷瀟泓皎洌瀏皓澡煌鑚鑽螽蛬臠臠截剪斫剴釿鑚刋鑽翦槎瓩竏倪蘗檗鞫窘蹙鞫亢竟覲菫饉衾噤窘箘懃忻釿掀矜鈞擒磬麇釁槿瑾麕听睾匱巍艤跂曦魏沂嶬嵬羲嶷礒瘧謔岌圄馭圉驍嶢徼餃澆僥翹蟯嶷憖釿岑崟狠沂圻垠听齦嶇詬于劬齲吼琥衢嘔吁懼吽栩枸窶瞿蒟煦佝箜杙弋齟齬懴懺餤茹噬啗啖餔柯岫齣閾羂縊傀儡絎杞枸艸瘡卉葷蒭芻蕘耨耘楔莽莽蔡嚔嚏藾鏈瑣鐺齣餒茹梳籖籤梳匳嚏嚔籖鬮籤麇麕橈摧擠撓衄擽熏燻燻熏頽粃秕頽屎摧韲齏麋瀉嗽漱梔嘴觜喙脣倔厥崛鞋鞨鞜泛襪韈勒銜啣湫諄縟絮圀櫟栩櫪椚檪頏瓔鉗箝枷軛剄馘踵踝跟刎縊縊陬暈澳嵎鶻抒冓辮斟翳熏燻廩銜啣祚杳芒儚矇暝曚溟眛瞎濛黯瞑罔啗啖鞁眩眩暈啖啗刳狡猖獗癲艱荼艱窘阨踝轂俥輛輌輅轎眩郛榑臘眩暝黝緇涅犂苴犁黔緇弋黯黝黎盧驪驪銕秬緇麁麤糲涅黔緇黝涅钁耨啣咥銜縷軋餤蕈椹熏麇麕皸皹燻裙葷醺嵎禺懼吽颶麌劬耦耦藕嵎禺懈痂褻蹊薊筓挈痙瓊綮剄冏盻枅逕醯蟪偈勁笄磬絅髻蹶愒謦奚竟禊烱挂硅勍煢炯脛彗檠黥奎迥夐閨袿谿旃黷渫腥褻穢褻黷覡鵙鶪鬩郤檄洫芒嗾銷刪删刋刮纈吝碣頡齧偈訐獗刔蹶髻孑竭鴂鴃囓桀楔譎蠍夬闕歇挈覈襭纈拮厥抉孒觜貶鑷毳烟烟烟毯氈烟烟欅螻鳬鳧蹙嶬嶢隘峭嶮嵌崚崢嶇巉嵒艱岑嶄隗幵嶮顴諠惓搴娟慳掀鉉繝蒹箝譴狷臉綣丱悁虔俔愆姸暄蹇筧痃羂慊拑甄涓夐眷蜷黔謇鉗蜆騫妍歉愃腱瞼壎枅鵑礙偈倪霓鯢猊黥囈鮨貎睨麑鷁鵙鶪鬩郤檄覡闃鴂鴃糱孽櫱蘖齧孼孑囓衒阮螈姸痃儼芫眩偐呟妍愿訝鉉繝估沽夸詁鴣瓠痼凅冱踞扈瞽涸胯觚蝴辜杞滸滬賈蛄罟倨沍炬怙呱琥蠱箍餬刳稠礫冀覬覦冀汞鱟亢吽磽滉搆膠扣桄扛鴿纐耿鏗靠曠盍蛟絎徨冓槔晧蝗餃隍肛皎纊傚咎撓洸篌狡杠窖遘胛闔哄煌蚣寇哮慷哽粳淆槹頏栲覯遑枸溘肓黌鬨矼簧嚮絖槁薨悾絳睾敲箜槓杲篁蒿佝羔倥恍哈摎媾誥烋夾狎惶咬鵁昿訌壙嚆伉犒鎬詬逅閧呷姮觥皓嫦鍠羹缸吼冦洽胱鰉吭鱇昊篝閘熕猴堽苟匣丐筓笄犢糱糀楮鸛芬顱趙腴踰逾跋沍冱凅蛩蛬鏐凩梏轂剋斛圀哭槲楫榜莓蘚苺茲粤于茲爰焉腑逞濛霎輦榻牀轂拵丐鐺抉拗踰沪逾濾狡刮檪櫟揩杪齷刮荅谺鯒兀汨榾笏矻鶻鐺鏝箏縡筝咸竭殫殄喙糝糂麭捏鮗恁鞐匳羔媚諂媚贅瘤瘻艚艀齣縷軋綢孅拱拱逕蹊腓籔禀稟罩厮廝閨怺焉輾虔誅弑戮戡殪衲衫諢滾褌鯤崑鯀很壼鶤齦棍焜溷菎蒟渾跟衮袞狠悃冱晤棊齬麌忤牾鼯唔圉沍蜈圄篌茣寤栲囂螯鼇耦慠哈咬熬嗷吽敖盒毫遨蓙兀楸鱓鮴懃諢嗄炸做蹉扠瑣鮓岔梭釵簔苴槎簑鯊靫娑莎柤嗟渣搓磋腮韲齏纔撕曬猜釵擠縡儕簔焠崔顋簑躋蔡倅灑靫齎綵砌犲伜眥滓霽淬洒摧賽豺寨臍眦鰓骰賽骰嘖禧烋祺祚閼徼壅遏囀哢啅冱棹找棹嶝坡陂圻陲畛疆竟垠徼垓悧俐黠盞觴巵卮觚罍泝嶝飫讌牾忤藹熾奘懋曄卉翕殷驕弉晟熹曩畴嚮疇曩蹕槭柞醋嘖筴愬齪簀鑿筰槊縒炸做拆炸刳撕劈擘磔屠遉壜罎嚆嘖喊拌辟躱炸髦貶逧礙囁聶筅簓緡蹐麾珥縉夾麾匕劄鍼扠箚螫遉遉蠍蝎嘸掟奠剳紮劄箚扎颯蔡楮鈔芫扨偖扠閭黠嶷蔗譬惺聆寤諳蛹絛磧銹寥愀寞銹礙逍徘徨徊彷佯冱洌淒滄冽凊凛凜鰄寤褪苻莢盒浚濬攫渫晞曬梟猴髏髑沛睾隰躁鬨鬧噪譟閙閙躁鬨鬧噪譟醂椹鰆牴芟攅竄盞簪戔纘孱巉槧鏨衫閂潸霰粲篡懴鑚懺嶄儳繖鑽爨汕蔘跚潺糝讒刪饌纔删驂簒糂纉攢疝鯢丗卅薺榴襍笊懴懺嶄儳慚慙讒塹竄戔巉槧鏨恃厮駛嗤駟廝嗜錙痣沚豕泗齎鷙廁啻祠耆眥滓翅巵畤咨鰤鮨竢卮蜍枳鯔塒嘴幟侈尸貲朿蓍眦茲徙縒鴟疵祗苡鰓輜祀魳址鉈瓷孶贄屎阯俟弑趾蚩厠耜緇舐撕梔謚呰虒諡熾觜粢揣鵄孳篩咫肆灑弑弑憖粃秕詆罔誣鹵胥醢滷鹹鹹鹵瀉栞撓聢尸蹙嚬顰俞兪呵咄詆訶咤嘯誚厠仄廁閾甎甃甓櫁軾壼樒梻茵荐仍鷸溥衍藉蓁稠孶葆阡孳薈茲痼錏錣鐚逡貎猊醢榻蜆寞闃謐澹恬寥惺蕭瀝涓淪湎湮汨簧咤咄蹤跟婉扈徇霤瀝漓褌掾朶騭隰嘯虱桎瑟蟋蝨躾粢茵蓐衽袵褥婉撓娜嫋娟冉隕薨歿殪殞徂斃筱鎬箘筱誄荵荵蕘篳霎荐亟驟臾紲絏梏縲縻痲痺痿痲瀑慳纐纈纈嶼蠹蠧犂犁滲沁覿笞僮俾閹奚臧墅偖赭鉈蔗炙洒娑嗟瀉灑畬奢鷓麝藉畭嗄妁癪鑠繳爍芍嚼蹐綽迹斫笏噦噦鯱鯱噦喃鶤髑髏炷椶陬芻溲掫麈啾繻鬚茱侏鄒銖娶娵棕蒭愀溲掫褶嗽啾甃逎泅菘鞦帚脩湫讎讐娶緝螽銹售驟聚楫遒皺慴隰岫箒鰌綉楸舅蹙齪鬻蓿倏槭俶菽謖齣恤蟀朮卹諄蓴蕣逡詢荀洵笋鰆浚濬恂悛蠢惷皴蹲雋儁筍吮徇蹠蔗俎疽砠苴抒雎墅胥舒絮杼梳沮咀齟詛蔬嶼蛆炒烝檣妝簫蚣湫猖筝踵鷦慵倡諍摏劭椒觴庠鈔餉霎璋楫鱆剏猩剿悚薔瘴慴蕭搶勦慫憔瘡陞淌樔鬆歙筱嶂鱶銷箏笙鮹韶聳誦牆浹稍艢刱悄漿椄襄呏懾愀菁敞顳拯殤霄頌囁睫逍瀟橦鏘誚廂聶舂燮翔樅旌嘯牀蹤腥竦邵蹌峭墻薑笙簫厠喞蜀仄軾稷禝贖矚廁穡謖寔嗇惻昃薔驃粲覈鞫蝨虱臀鞦紂卻屏屛逡黜闢擯屏屛鐫卻貶肛漿瀋幟讖箚劄皓晧皚皙皎鐐鏐帛堊潘皺皴襞吝嗄謦皺撓臻畛嗔駸鱏沁襯譖瀋袗蔘僣忱箴糝椹譛蕈鐔瞋怎蓁滲齔斟鍼僭縉罧岑讖宸呻贐簪晨蜃軫脣哂抻潭鷆籡塒邇弍怩茲轜珥峙瓷孶膩輀孳臑雉恃畤忸舳衄衵昵麝闍鰙鶸鵲搦蒻麝儔銖誦繻臑聚頌蠕襦孺顬戍懦狃絨揉蹂鈕儔糅忸鞣孰朮卹戌恤荀洵笋恂筍徇諄蓴詢鶉蓴茆蜍舒絮茹杼耡抒繞滌驤鑷饒襄嫋迢顳拯橈攘囁聶弉仍蟯諚嬲烝躡奘蕘掟絛遶仗禳掾溽褥縟蓐椹蕈蕁燼潯贐恁蜃姙袵糂荵衽仭仞紉櫁樒芻麈籔鬚蒭笊鬆簀樔醋醯跿跣惴雖隧忰膵邃綏燧騅捶彗萃陲悴祟崔瘁觜榱陬蒭皺鄒芻菘呷嗽吮裔甄杪饐賺眇眇攀縋耜犂銛犁耡郤郤罅釁軼邁犁梳耡耨犂樔拯抔樔汕贍歉尠竦蝎犖桀雋儁乂髦儻俶瑰毫稍淒槊苆凛凜淒鮨鮓腱綫芒澣浣滌洒漱盥澡酳餤陟迪廸餤侑聳饋烝酳慫慂歔啜欷哈酳歃歠啜鸞鑞鑾鱸凊菘裙裔魍魑魎鼈拌擲捐已愿樸輒輙脛骭臑拗簀簟剽勦慓昴闔渾馭攬綰辷歙澂嵎陬亟倏遽蹶菫澂搨揩掏鑢擂銛兌鏃鯣荳隋惴隧螟龕瘋桷狡丗睛洒凊穽橇砌筬眦鶄淒霽晣菁蜻倩聟撕薺丗臍旌晟腥惺掣毳韲嘶蛻齏齎擠晢筮儕噬躋眥猩悽贅繦襁繈忰倅悴伜躮跖炙勣蜥皙瘠蹐晳槭磧迹淅藉鶺裼蹠蹙螫晰齣蓆筵喘嗽謦齪喘嗽跼褻緤梲薛楔渫椄掣截晢啜紲浙絏歠鑷泄晣膂阨陜褊湫陋隘窘拮逎遽遒蹙蜩鬩訶譴誚謫誅譏糶暹譫鱓霰譛篡倩荐蘚筅簽涎僊摶僉顫羶仟韆亶氈僭潺塹璇擅癬澹銛翦嬋饌簓阡濺旃籖闡綫戔雋蟾吮殱尠孅殲栫痊簒愃槧笘疝餞沾芟洒剪磚銓譖筌跣喘陝籤刋甎孱瞻燹僣贍牋臉鐫疝痳甅竰僊毳蛻筮噬贅蚋橇蜹嬋蠕喘贍髥冉髯荐涎擅薇徂怎愬麁麤梳沮咀甦胙耡齟詛蔬酥蛆泝俎疽砠醋祚胥哈瘡帚艘樔鬆丗筲箏抓輳卅鮹愴弉嗾稍漱怱刱魳躁錚懆粽滄艙囃棗簇澡鏘廂噪箒悤奘叟娵颯牀找蹌棕薔譟慥皁嫂炒笊艚妝偬溲掫筝贓腠諍怎崢嗽淙艸蔟鐺孀鈔臧匆霎椶剏剿籔搶勦歃嬪筝箏驂驂枌簇矚嗽蔟熄惻昃喞仄鏃翦礑臀蠧戔銷蠹蠱謗誚呰疵讒譖譏詬誣詛貶謗譛詆灑洒淙瀉盥溲瀝濺喞澆嗾猝伜倅饌饌峙奠埆猜囿厥瘢峙屹仄崛聳欹竦聳聳竦杣剌乖辜詭涅嫋昊霄旻諷諳誦毳轌橇艝剔厥忖栫拵洒蹲邨奘慥襍艚贓臧弉蔟鏃簇侘綏岔佗駝荼朶跎躱鱓粏鴕咤沱隋鉈咫蠆靆棣薹頽駘詒抬玳褪蒂鐓蔕擡兌紿颱瑇颱苣炬幵栲稷殪猗綽嫋婀娵仆斃蹶僵沛殪倬崚崢嶷敞兀穹巍襄亢嶢岌崔嵬崟魏睾杲亢簟篁葆齎裹貲箍佚繆靠鏨鑚鑽綰耡瀑蕘滾磔戳謫棹拆柝倬啅魄炷逞逞綰薀蘊峙疇畴伉綰篁蕈簍笆悍驍赳筰闌酣笋筍筍笋篁猖闌哮獗胼鱆胝腁鮹慥嗜耆嗜窘贍襷伜裨倅幫拯掖幇掾翊侑裨找繹頌鬮鬨敲敲擣搗扣站竚佇鑪祟祟祗啻謇糺飭尹鞫逕啻飄漾爛靡糜爛闥佇竚溘驀倏躊踟裴躑躇撻獺燵哳靼梲怛闥勦釿殄蹶遏謖截站剿羈桿杆鹵髦鬣榜蓼譬譬俔壑谿峪猯熈聊酣僖煕槃佚熙懌豈怙恃馮憑倚莨迸逬繃羈襪韈羇羇羈髻誑誑髱瑶瓊璋釐賚鐶魄賚渟氓癬戍喟躊踟逡躇檠揉撓葆袂聊盥槃誑贍榱桷椽孰髫髦綏囈譫橈撓諢詼謔婬撓澹痰袒鐔疸罎憚靼殫酖啗鄲怛站譚潭襌檐憺餤湍緞賺壜啖蕁摶覃赧毯眈猯蜑褝亶攤荼朶兌鴕懦梛沱隋儺拿娜糯拏駝抬餒擡棣睇薹駘牀蔔橙搦鴕獺怛妲蟎賺瞞騙懈畴疇喃譚緞赧煖胝躓雉輊黹褫黐魑篪踟笞峙瑣藐幺杪眇稍咫詛邇詛昵膂莨遒逎杠杠舳苣苣胄帙膣蟄腟鵆釁孩孺粽閧衢茗謫躇擲誅狆冑鍮鈕偸綢蟄畴儔紂籒稠誅疇躊惆籀胄籌朮黜紵竚躇佇楮杼糶輙貂鰈梃澂掉笘听沾漲蜩鬯疔稠昶冢啅幀髫雕窕輒誂褶齠趙晁悵渫佻迢蔘飭陟躅灑埃嵌鏤銷渙酖抻鴆椹碪闖趁狆縋屛屏竟訖啅糜棍枴梃塿塋壟垤冢鋏痞閊宦痞宦揆衙尸鞏攫拿拏敝儡芒劬瘁羸憊樛蟾鴾坏扈殫歇泯竟殄銷涸竭擣膠馮舂搗憑橦鏗傅樅戳搶几殱殲殄銷竭殫捏蹲蹲旁甄衲纉纘弍踵椄鶫鶇箝拑噤淹跟柎誥蘿臚蘿鐓霾霾壎砠恙傴矜蹙竦愨兢虔恪劼聳愿悛祗齪飭堡坡陂裹苞苞韜葆裹輟襤褸髱苞苴孶懃孳勗劼劭黽仂邁勖閔懋攣絆絏縲緤羈綰縻紲彜彝抓莪鐔翅顆礫呟瞑坩蕾莟蕾褄帑孥跌躓跋蹉蹶竦翹跂襭孰鈕拈抓辜謫蘊抓剪薀緝飄猋颶飆冽釉驍勍驃梟驕倔伉逎勁遒倩瑣繹肆繹鉉攣鋏逑儷仗賁劈擘弖腆嚔酊牴啼詆髢洟叮幀裼蒂柢裎蔕遉棣睇蟶蜓掟酲觝嚏涕騁剔渟梃赬逞听羝霆釿梏柬牘牋緘逖迪糴剔躑俶廸擲覿狄輦梃桿杆槓柤檻櫺闌銕耋啜佚錣垤輟咥餮跌衒壥廛輾殄蜓椽忝鈿覘躔靦唸諂畋甸恬簟巓癲鷆篆囀嗔癜奠霑碾腆沾貂遘佞濔瓧籵ー竍滌疔儡瓰竕涅捏畋甸黏鷆臀癜奠拈碾輾鈿抖肚蠹屠跿蠧蚪闍荼覩睹厲箚鞳淌閙鞜櫂鰧韜棹恫啅叨蟷蚪螳骰滔饕俑縢盪艟掏罩纛棠竇嶝疼鬧瞠橦揄夲蹈鐺鶇僮絛慟滕橙礑籐擣剳溏磴檔幢搨抖螗鼕劄榻儻蘯搗襠荳鍮苳綯撓掉荅偸帑薹聘娉詢炷菘蜀鶤夐迢邃迥遐藐逖茫浹奎銷煬鑠爍謫樛辜咎枅櫨譴咎咎譴鴾晨鵇鬨纛竇髑黷牘犢慝梳詁厲塒渙鑠銷蒂蔕棘朿牀輹祚茲祀旄耄艾耋叟耆蟄緘翕噤鑰闔鉗縢緘杼鈕咄柮吶訥迚迚飭湫紮渟淹輟遏誦徇倡幃幎帷幄鵄鴟矼翊疱闔翔騫蜚恍匱篷蹇渟遏舳艫鞆儕們儔檠炬炷檠纜偕塒拿拏緝擒圄罕嗇穡穡樊菫擒俘羈堡寨黐俘搦弋撈搏攬捫簒搴篡秉觜盪蘯褪暾燉飩遯瓲弩孥帑駑呶棠橈鬧瞠蹈耨僮慟曩幢呶獰鐃檸瑙撓衲閙儂恫臑艟襦髑螫顱鰌鯲肭吶褞鴿蚌醪謇吶訥哄鑼錚鐃濘淤緞嫩飩壜遯罎杼梛儺拿娜糯拏蔬茹毋罔綯秧蹇痿仍沚幇夥幫毋霖潦霪脩曼夐轅袤眄倪睥睇謫眄旒漓疚痼妣梛沚涕唳哭嚶呱喑啼喞啾挌擲抛拋擲愾欷唏嗟吁喟慷慟嗚咨拋抛妁眷昵駮做薺僣僭鉈屶皙棗拊捫抔奚曷靡靡柬嬲鎬葷腥羶懈憖憖鱠膾韲齏膾癜鯰魸嬌訛譌譌訛瀾泪涕洟泗瀾靼韋鞣鞣膩吮舐墅艱懊疚蹇艱猗枹肄嫻嫺傚狃狎幷騈伉弍并儷駢臚駢驪騈肆麋忸褻狎諳泄狃綯縲煖喃曷盍鉅奚膩邇弍瓊贄錵鳰鬯彷彿膠膠荼滷皰靤殷獰輜竄逋遯毳醪溷渾淆綉鰊鯡霓滲躙躪偐偐蠡蜷蝸鞅佗贏駑銖駘鮸蒻茹鐃仍遶繞橈饒薤淬焠俾睨眈眥睥眦盻睚燗臑髴髣爛榆楡枌棣遽霍驟倅猝溘伜瀑棣潦恁姙袵荵衽蔘孥駑黻黹綉黼黹絎鵺粳倥濘濘蹐鑷扎揩蛻帛睇偸攘饅蓴茆袍絖釁栲寃冤涅柢嚀佞檸濘歛覦僥覬犒犒塒捩拗捩悋猜筐筺榻牀涅捏黏舐趾蒂苞蔕柢閨寐拈冉輾黏鯰綣慇殷嚀叮諄懃瑙衲儂曩喃薇逋遯佚竄宸櫺梠霤簷檐廡芒詒貽燼畸熨熨闕耘撥覘抒覬莅瞰歛覦曰臙吭嚥亢頏詬詈壙抻舒燹窈贏覃莚鬯昶舒抒聿陞陟躋幟襄騭躋躡陞陟鑿已鑚鑽漿歠嚥范彜彝餬笵驀驀馭麇麕駑詛詛燧烽嫩玻耙爬菠怕袙坡霸陂哈笆叭葩跛霈珮胚碚裴焙沛癈擺悖琲佩憊坏怫孛旆湃徘鷂跂爬匐匍鰙鮠塋儚儚阡銓筴籌謨揆銓揣籌謨忖癸仭揆仞咨鈞詢齔脛骭怕雹駮樸璞霸帛珀膊薜搏魄蘗檗陌佰擘嘔帚彗喀躡箒哇佩瀉褫齦齶孚莠厲勗勖厲匚匱篋匣筐櫃筥筺鋏螯扠翦拑剪夾鉗筴箝鋏觜筴嘴艀鷂婢楹迸驟逬駸猋犇蹌驤慚慙愧詬薑椒炸胚弌俶剏刱瞞赧愧怩忸靦慚慙藕戮詆詬忝騁駛櫨鯊櫨幢旆旛旌罕旃旄旒螽旄霸疥杠橦綮礑鱩鰰纛矻拮臚褂衫襯褻裙袢襦跣跿袒裼撥盂甌礑魃叭撥跋垓鴿洟葩嚊咄譚莟泗洟竄蕁縹縹衄很已嚔嚏葩瓣萼餞贐翅葆槔槹撥刎媽柞憚憚鎺搏沮扈瀰衍滔莚蚌莎薛薹魬薺嵌嵌鱧鰙鮠湍猝簪驤鴥駛飄偈霍鴪驟囃囃湍颯鶻肚祓禳祓禳禊攘擽蘯箒盪襄鞅鰚鯡蛔匐匍姙孕胚鯡腑箴鍼磔桀辜蝟渺眇迥夐遐藐逖茫迢杳瘤疽瘻癰霽肬肛疣繙攀燔飜笵胖魬鈑樊潘蟠鷭絆拌袢膰槃礬瘢范旛泛蹣槧楾瑪蟇痲蟆碼麼玫眛苺黴邁霾莓狢藐瀑貘貉貊驀陌獏擘蟇駮寞蟆奕撥桴捩枹襪袙秣跋茉靺魃薔蟠輓鬘鷭絆袢槃礬謾曼卍縵旛饅蹣瞞幔鏝魬鈑妣脾狒翡痞贔辟砒賁琲羆坡裨蓖陂鞁怫痺匕屁鯡靡榧粃臂蜚鵯髀糒跛髴譬俾朏丕嚊鄙秕婢菲霏紕鞴腓痹曦梭燬杼贔屭屓燧扣曦釉暉耿煕烱炯皓熙耿爍熈暉鰉輾磑蟇蟆蟾癇攣痙臾掣弯拏碾彎輓搆曼轢攀扞掎覃拿湫矮羆蜩衒鬚髥髯蘖肄孽孼櫱籤籖楸鬻瓠蠡匏梠簷廂檐廡淹曩黻跪醢犇璋臂几痂疥癬嚬顰嚬顰脾鶲淹溲涵沁襞褶饑篳挈鵯蹕篳謐櫃匱椁槨柩轜輀旱魃弌褝袗衫襴襌俑鈞弌朞睛眸囹圄圉孑煢俚鄙鄙捫拈閘熨暘狒皸皹皴罅韵籟嚮郤遑嬪纓繙胙燔膰佰襞飄縹殍猋飆慓鮃馮憑怦嫖驃凭鰾剽苹雹柝愎鵯鵯鐐闡昜辟擺攤闢拆闢豁扁鮃飜翩飄飜昃仭仞闊旁敞瀰衍熙泛茫溥熈豁滉昿瀚汪淹煕曠扈曼濶袤衍闡鶸鬢嬪榀禀稟殯擯嚬檳顰蘋繽寐靡濔縻糒鮇麋薇糜嵋鞴瀰媚黴弭鐚樒謐櫁闢繆杪萍屏渺藐屛眇弸緲罠黽旻檳顰緡愍憫閔紊泯鬢壜罎榑桴趺脯誣拊黼孚艀賻蜉巫餔罘苻孵枹坿麩郛俯鳬俘鳧仆殍腑逋麸傅柎溥咐俛呎ー鑪鞴鞴艀馮罘瘋諷籟鰾簫龠籥笙簧鱶窕穹汪覃邃窖泓浚濬潭澳苳箙匐愎鞴袱蝮蔔茯鰒馥輻輹蝠咐呵吩濆袱匏瓠銜啣腓梟鴟鰒酖畚淤哽壅隘堙阨噎湮堙怫雍閼壅栫罧罧芫仆俯偃熏燻麸麩衾沮扞圉捍篝偃俛俯縒盒弍弍盒牋榜笘簽扁槧牘褻衵禀稟潯泓稍潭廩沂彿髴黻祓怫聿毫艙艤舫艘篷舸帙帙蹂轢躙躪躡躅跋迪躇躔蹠跖踵廸藉駘蹈蹂阯梺掉篩擲擺篩抖掉顫誥檄牴觝袱刎芬氛吩枌濆忿賁褌嘸廡錻巫坿鶩憮柎艀毋誣鞴茯豕駮蚋蜹椈蚋蜹蚋蜹鰤錻紊刎屁萍鮃薜秉斃憊屏俾嬖娉迸屛逬苹聘幷枋炳睥并敝墻牆燹躄躃霹甓襞辟闢劈瓸粨ー竡舳艫臍綣蒂蔕鼈佞諛諞諂貶絎褊抃蝙扁諞騈腁辮騙翩胼貶駢麑袂汨幎覓冪羃鼈韈襪臙騈腁眄辮泯胼冕瓣俛駢抃湎鮸黽褓黼匍堡餔逋鞴咐葆脯焙澎抛娉苞龐匏迸硼彷彭魴篷麭枹舫髱膀磅逬榜蚫怦幫葆旁謗疱拋靤枋絣泛烽弸匚幇褓鉋抔髣鮑袍皰堡蚌焙滂琺炮蒡堋菠咆繃箒帚彗孛彗耄焙楔堋抛拋咆吼哮臉髥髯顴佗仆蹼樸袱痣黶鉈槊棘戈戞戛桙仗祠埃侘夸矜糒擅驕蘯亶盪肆侈脯脩膊晞毫臍榾榾絆孛甌鵑迸逬滸沂濆潯陲骼恍髣彿仄洸髴曚仄諷仄檣艢屠斂麁麤頌嵌蠡洫塹隍塹勒鐫雕鏨袰輜泯糜湮淪剪誅剿勦翦殱殲笨賁繙犇飜畚謨拇媽姆濛檬儚昴尨氓茆莽桙旄厖芒袤拇袍朦髦蚌滂眸蒡堋黽蠎懋魍鋩鴾曚甍惘罔膀耄艨瀑矇榜茫蟒旁謗棍孒暈暈鶩沐蹼繆樸苜暈鈕孛悖歿鯔襤褸縷捫梵燔惘听磅嘛碼麼蟇蟆痲莓玫眛苺瑁邁霾臧衙眩樊籬寨篳枸枉佝樛紆弯彎圉寞幔繃秣芻蒭秣耙髷枉紆僂枉橈洵恂諄愨詢忱悃孚亶寔苟洵悃鍠鉞戉薜猴糅蠱渾厠淆糂糅襍廁尨媾崔枡桝斛坿茲櫨枅茲窶籬襍椏胯奎胯胯恂駮犂犖犁襠闍陌竢驀秣茉靺俟佇竚竢睫烝祀祠畤奠祠餽祀躔綢紆繞摎樛繆繚蟶蟶紆繞纓裹繚綢牖眩欒蠱俎眦睚眥庠黌聘眩蔟眩瞼猯覲蝮菽胝荳仗捍秉戍麋枋駮鋺毬圜摶欒髷摶罕迴卍縵饅蹣瞞幔鏝鬘蟎謾曼懣卍麋黴弭靡濔躬眷澪瞰甕朏瑩磋擂砌瀲濆潭沚潯滸輅覡巫誥孕姙雎鶚侏矮蹼躬汞薔蜃蛟癸髻鬟肆壥廛丗卅禊鷦齠齔臠洫竇壑霄霙猾紊淆梏嫖猥婬猥叨惷攘訌悖蚩紜飆撓樊駭撩猾溷紊廸陌揆馗衢蹊隧逕阡迪牖迪廸擯弸濔櫁樒褌瞠僉胥咸漲鏖蚩莠岫巒岑椒崟嶂簔簑蜆瞠薨矍邏邏馘蚓珥嫻嫺瞑藐眇螟暝茗瓱竓瞰覿覩瞻睹瞿眄覯矚甄泯緡愍憫閔罠旻憮眸鴾毋嘸嚮訝邀畴疇噦縢嚮尨疇侑畴讎讐毳蕣槿倩聟鼯餮叨婪饕溽齲蠹蠧毟挘蒻蓆莚筵貉狢餾烝綰紆艱噎饐哽噎贅捶笞敲笘笞撻捶榜繦襁褓繈鞅鞁纓桴豁昿壙廖曠隍肓膺匈邨蔟麇麕攢攅簇蔘蝟榁窩鰘瑪碼溟螟暝茗瞑酩鯢眷卹煦恤繚遶繞迴圜蟠樛遶徇邏繞輾迴圜徼紆匯躔槃繚浹厮廝糝盧辟聘畸瑰禧娶聘娉筴筮蓍蓍鮴暈眩俛湎瞑泯眄懋媽姆謨麼裙朦髦蠎魍鋩曚甍惘罔耄艨矇茫蟒濛檬儚氓莽旄贏劄箚禀稟毯氈燼燼皰疱苜沐槿朸艾鼴鼴儻綟綟捩裙鴂鴃鵙鶪罍罌缸瓮甕抬擡齎凭靠懣黐粢擡抬朮糯齎畚簣摶亶縺縺丕阯髻僥徼邀覓怫乖繆狠很忤剌愎悖蛻蛻慵懶嬾懶魑魎魍樅揉搓揉脾髀靄舫舫糱渫泄傅銛沮泄醪烝黎瞞懣捫們闕闥椰鵺捓揶墅歟輻囂諠軈畬畭葯龠籥檪鑰櫟隘軛扼阨奕燎燻炮烙煬燔焠炬炷燬熏炙廨燬椰壥廛豢餔頤鏃扠綏恬歇綏謐祺晏恬憺鑢佚綏恬埆瘠羸箘萢窶悴窶瘁忰憔萃倩簗箙吝嗇悋硅敝敝陜痒痾疵豺犲陜轎疚棠巒蕷谺僊疚訖痒鰥歇已厲熄歇弭偃輟鰥矜釐孀鰥稍臑毳雍怡廱煕豈熙熈緝燮碼ー踰渝逾揄俞臾兪榆楡瑜萸覦腴鼬雍諛蝓瘉脩肬牖蕕囿蚰鼬蝣侑悒莠疣游釉黝餔餔榻牀夬裄趁靫徂邁于韜杠擅攘腴愃胖饒殷饒檠榜茹弭溲膀窿韜撼鬆撼侑綽頌攤縵舒唹淤蕷畬畭舁歟飫丗晨俶臧懿价煬曄纓踴頌咬窈徼邀姚瓔榕暘怏癰杳瀁窕佯慂雍徭燿壅鷂殀漾靨膺恙瑶癢慵昜孕夭痒廱臾俑幺蛹拗茗酲酩醺酊稍衾薏峪杙閾弋翊贅讒奸辟佞慝睇滓葭芦蘆媾攀捩妝妝鷆涎仍胄衢淅謚諡掫詁甦籀詁誦籒娵嬪艾苹蕭萍蒿縒仍凭藉縒搓仗倚靠拵旁馮仍憑冑葯苙禧驩煕僖熙驩懌熹熈懽忻兌讙怡蹌蹣羸懦孱羸蘿蝸騾鑼喇邏蠡厲磊擂藾蕾儡癩綟糲賚罍癘籟醴擽犖駱珞烙駱駝埒薤糲溂埒蝲剌喇騾懶婪纜闌癴籃襴欒巒爛鑾嬾鸞瀾燗襤欖攬黎莅漓驪犂黐釐籬犁罹蠡茘蜊悧俐詈俚莉仂朸篥勠蓼戮篥珞擽霤瘤犁瀏苙嚠旒鏐游榴窿餾犂鑢沪閭濾廬臚櫚梠絽驢膂廖繆聆鏐輌倆魎羚壟聊蓼撩粱繚寥櫺鬣裲喨踉楞隴鐐燎鷯崚輛鱲仂籙朸禀霖稟藺凛痳廩凜躙淪躪悋懍驎吝醂侖綸痳塿鏤僂螻瘻簍縷鑪褸璢廬羸縲瘰誄泪堝坩藜蛉綟鱧儷櫺蠡茘癘鴒犂醴莉唳黎捩澪糲聆厲犁驪囹羚瀝珞櫪靂癧擽礫轢檪櫟冽捩洌縺薐匳斂賺輦臠蠊瀲楝鏈鰊攣癴櫺臚盧顱滷輅櫚梠絽鑪驢髏膂轤鹵廬櫨鑢沪芦鱸蘆閭艫艪濾哢醪壟蜋鑞朧簍髏縷陋蘢螂踉楞隴褸窶檪瑯櫟塿鏤龐薐僂撈臘螻潦瓏榔癆瘻莨琅轆烙仂勒碌朸鈞驢淪崘崙侖哇萵鐶薈矮猥匯穢夭嫩鰙殤夭殀衢綰腋掖滕腋掖孽孼蠖框洶濆滕詁夬揀髷鬟倡孼夭殃厲慝孽氛雕儂銖毫錙纔臾涓纔縟囂絮絖纊袍纊絮竟躇蟠罠羂侘佗佗侘侘佗蒭芻稈哂听呵嗤咥唹粲僮拌秕匈慝獰粃穢猾狡獪黠棍酲儂鋺蜿綰弯彎盂"
      }
    };
  },
  computed: {
    rectParams() {
      const size = 300;
      switch (this.hiddenDirection) {
        case "上":
          return {
            x: 0,
            y: 0,
            width: size,
            height: (size * this.hiddenParcentage) / 100
          };
        case "下":
          return {
            x: 0,
            y: (size * (100 - this.hiddenParcentage)) / 100,
            width: size,
            height: (size * this.hiddenParcentage) / 100
          };
        case "左":
          return {
            x: 0,
            y: 0,
            width: (size * this.hiddenParcentage) / 100,
            height: size
          };
        case "右":
          return {
            x: (size * (100 - this.hiddenParcentage)) / 100,
            y: 0,
            width: (size * this.hiddenParcentage) / 100,
            height: size
          };
      }
    },
    kanjiList() {
      let str="";
      this.activeKanji.forEach(key => {
        str+=this.kanjiDict[key];
      });
      return str;
    }
  },
  methods: {
    nextKanji() {
      this.boxVisible = true;
      this.currentKanji = this.kanjiList[
        Math.floor(Math.random() * this.kanjiList.length)
      ];
    }
  }
};
</script>


