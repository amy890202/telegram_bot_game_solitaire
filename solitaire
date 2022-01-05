from telegram.ext import Filters, Updater, CommandHandler, MessageHandler, CallbackQueryHandler
import random
import copy
from telegram import InlineKeyboardMarkup, InlineKeyboardButton
random.seed( 10 )

with open('token.txt') as FILE:
    token = FILE.read().strip('\n')

updater = Updater(token=token, use_context=True)
dispatcher = updater.dispatcher


class Card():
    card_to_name = {1:"A", 2:"2", 3:"3", 4:"4", 5:"5", 6:"6", 7:"7",
                    8:"8", 9:"9", 10:"10", 11:"J", 12:"Q", 13:"K"}

    def __init__(self, value, suit):
        self.name = self.card_to_name[value]
        self.suit = suit
        self.title = "%s%s" % (self.name, self.suit)
        self.value = value

    def isBelow(self, card):#判斷牌為接續的
        return self.value == (card.value - 1)

    def isOppositeSuit(self, card):#判斷紅要接黑 黑接紅
        if self.suit == "♣" or self.suit == "♠":
            return card.suit == "❤" or card.suit == "♦"
        else:
            return card.suit == "♠" or card.suit == "♣"

    def canAttach(self, card):#如果以上兩者皆成立就可以接
        if card.isBelow(self) :#and card.isOppositeSuit(self)
            return True
        else:
            return False

    def __str__(self):
        return self.title


class Deck():
    unshuffled_deck = [Card(card, suit) for card in range(1, 14) for suit in ["♣", "♦", "❤", "♠"]]#產生四種花色a~Q的牌

    def __init__(self, num_decks=1):
        self.deck = self.unshuffled_deck * num_decks
        random.shuffle(self.deck)#洗牌

    def flip_card(self):
        return self.deck.pop()#pop出一張牌

    def deal_cards(self, num_cards):
        return [self.deck.pop() for x in range(0, num_cards)]#pop 很多card 用來做開場分別在7個Tableau依序pop1~7個數的牌

    def __str__(self):
        return str(self.deck)


class Tableau():#7疊牌
    # Class that keeps track of the seven piles of cards on the Tableau
    def __init__(self, card_list):
        self.unflipped = {x: card_list[x] for x in range(7)}#開始妻張都沒翻牌
        self.flipped = {x: [self.unflipped[x].pop()] for x in range(7)}#開始從第x排未翻排的牌堆中的pop一張翻牌

    def flip_card(self, col):#翻一張牌出來
        """ Flips a card under column col on the Tableau """
        if len(self.unflipped[col]) > 0:
            self.flipped[col].append(self.unflipped[col].pop())#從第col堆翻一張牌出來 從unflip移到flip


    def addCards(self, cards, column):#判斷如果可以移就移卡過去
        """ Returns true if cards were successfully added to column on the Tableau. """
        column_cards = self.flipped[column]
        if len(column_cards) == 0 and cards[0].value == 13:#K可以放在沒牌的位置
            column_cards.extend(cards)
            return True
        elif len(column_cards) > 0 and column_cards[-1].canAttach(cards[0]):#其他只有can attach的檢驗通過才可放在有牌的位置
            column_cards.extend(cards)
            return True
        else:
            return False


    def tableau_to_tableau(self, c1, c2,index):
        c1_cards = self.flipped[c1]

        if self.addCards(c1_cards[index:], c2):
            self.flipped[c1] = c1_cards[0:index]
            if index == 0:#如果是翻開最上面的牌(該排都被移走了 就翻開一張牌# )
                self.flip_card(c1)
            return True
        return False

    def tableau_to_foundation(self, foundation, column,row):#移到答案區
        column_cards = self.flipped[column]
        if len(column_cards) == 0:
            return False

        if foundation.addCard_row(column_cards[-1],row):
            column_cards.pop()
            if len(column_cards) == 0:
                self.flip_card(column)
            return True
        else:
            return False

    def waste_to_tableau(self, waste_pile, column):
        card = waste_pile.waste[-1]
        if self.addCards([card], column):
            waste_pile.pop_waste_card()
            return True
        else:
            return False


class StockWaste():

    def __init__(self, cards):
        self.deck = cards
        self.waste = []

    def stock_to_waste(self):#從未翻開的stock中翻出一張牌
        if len(self.deck) + len(self.waste) == 0:
            print("There are no more cards in the Stock pile!")
            return False

        if len(self.deck) == 0:
            self.waste.reverse()
            self.deck = self.waste.copy()#stock沒牌了就把waste全部拿回去
            self.waste.clear()

        self.waste.append(self.deck.pop())
        return True

    def pop_waste_card(self):#從waste拿牌出來
        if len(self.waste) > 0:
            return self.waste.pop()

    def getWaste(self):
        if len(self.waste) > 0:
            return self.waste[-1]#得到waste牌的value
        else:
            return "empty"

    def getStock(self):
        if len(self.deck) > 0:
            return str(len(self.deck)) + " card(s)"#stock(未翻開的)目前還有幾張牌
        else:
            return "empty"


class Foundation():#答案區

    def __init__(self):
        self.foundation_stacks = {"♣": [], "❤": [], "♠": [], "♦": []}

    def addCard_row(self, card,row=-1):
        if (card.suit == "♣" and row==0) or (card.suit == "♦" and row==1) or (card.suit == "❤" and row==2) or (card.suit == "♠" and row==3)or row==-1:
            stack = self.foundation_stacks[card.suit]#第suit花色的牌堆
            if (len(stack) == 0 and card.value == 1) or stack[-1].isBelow(card):#沒牌時只能放A 不然要isbelow才可以放
                stack.append(card)
                return True
            else:
                return False
        else:
            return False
    # def addCard(self, card):
    #     stack = self.foundation_stacks[card.suit]#第suit花色的牌堆
    #     if (len(stack) == 0 and card.value == 1) or stack[-1].isBelow(card):#沒牌時只能放A 不然要isbelow才可以放
    #         stack.append(card)
    #         return True
    #     else:
    #         return False


    def getTopCard(self, suit):#第suit花色的牌堆目前top card
        stack = self.foundation_stacks[suit]
        if len(stack) == 0:
            return suit[0].upper()
        else:
            return self.foundation_stacks[suit][-1]

    def gameWon(self):#如果全部花色都有放到value 13就成功
        for suit, stack in self.foundation_stacks.items():
            if len(stack) == 0:
                return False
            card = stack[-1]
            if card.value != 13:
                return False
        return True




def printTable(tableau, foundation, stock_waste):

    text = "♣(0): "+ str(foundation.getTopCard("♣"))+'\n'
    text += "♦(1):"+ str(foundation.getTopCard("♦"))+'\n'
    text += "❤(2): "+str(foundation.getTopCard("❤"))+'\n'
    text += "♠(3): "+str(foundation.getTopCard("♠"))+'\n'
    text += "🂠(4):"+str(stock_waste.getWaste())+'\n'

    for col in range(7):
        print_str = ""
        hidden_cards = tableau.unflipped[col]
        shown_cards = tableau.flipped[col]
        for i in range( len(hidden_cards) ):
            print_str += "\t*"  # 表未翻牌的牌
        for i in shown_cards:
            print_str += str(i)  # 表未翻牌的牌
        text += 'row'+str(col)+'('+str(col+5)+'): '+print_str + '\n'
    print(text)
    return text
    #
    # """ Prints the current status of the table """
    # print(BREAK_STRING)
    # print("Waste \t Stock \t\t\t\t Foundation")
    # print("{}\t{}\t\t{}\t{}\t{}\t{}".format(stock_waste.getWaste(), stock_waste.getStock(),
    #                                         foundation.getTopCard("club"), foundation.getTopCard("heart"),
    #                                         foundation.getTopCard("spade"), foundation.getTopCard("diam")))#waste,stock,ans區
    # print("\nTableau\n\t1\t2\t3\t4\t5\t6\t7\n")
    # # Print the cards, first printing the unflipped cards, and then the flipped.
    # for x in range(tableau.pile_length()):
    #     print_str = ""
    #     for col in range(7):
    #         hidden_cards = tableau.unflipped[col]
    #         shown_cards = tableau.flipped[col]
    #         if len(hidden_cards) > x:
    #             print_str += "\t*"#表未翻牌的牌
    #         elif len(shown_cards) + len(hidden_cards) > x:#比目前最長的牌組更長
    #             print_str += "\t" + str(shown_cards[x - len(hidden_cards)])
    #         else:
    #             print_str += "\t"
    #     #print(print_str)
    #     text += print_str+'\n'
    # #print("\n" + BREAK_STRING)


d = 0  # 產生一組鋪克牌
t = 0  # 先pop出牌面上的牌
f = 0
sw = 0  # 再pop出24張去stock
def func(update, context):
    global d
    global t
    global f
    global sw
    if update.callback_query.data == 'a':
        if sw.stock_to_waste():
            result_text = printTable(t, f, sw)
            #context.bot.answer_callback_query(update.callback_query.id, '你按的是功能 A')
            context.bot.send_message(chat_id=update.effective_chat.id, text=result_text)
    elif update.callback_query.data == 'b':
        d = copy.deepcopy(initial_state[0])
        t = copy.deepcopy(initial_state[1])
        f = copy.deepcopy(initial_state[2])
        sw = copy.deepcopy(initial_state[3])
        # d = Deck()  # 產生一組鋪克牌
        # t = Tableau([d.deal_cards(x) for x in range(1, 8)])  # 先pop出牌面上的牌
        # f = Foundation()
        # sw = StockWaste(d.deal_cards(24))
        start_text = printTable(t, f, sw)
        print(start_text)
        # print('start_text',start_text)
        context.bot.send_message(
            chat_id=update.effective_chat.id, text=f"{start_text}")

        # d = Deck()  # 產生一組鋪克牌
        # t = Tableau([d.deal_cards(x) for x in range(1, 8)])  # 先pop出牌面上的牌
        # f = Foundation()
        # sw = StockWaste(d.deal_cards(24))  # 再pop出24張去stock
        # start_text = printTable(t, f, sw)
        # # print('start_text',start_text)
        # context.bot.send_message(
        #     chat_id=update.effective_chat.id, text=f"{start_text}")  # update.message.from_user.full_name)
        # context.bot.edit_message_text('你按的是功能 B', chat_id=update.callback_query.message.chat_id,
        #                                message_id=update.callback_query.message.message_id)
    # else:
    #     context.bot.send_message(chat_id=update.effective_chat.id, text="你按的是功能 C")
def print_now(update, context):
    start_text = printTable(t, f, sw)
    # print('start_text',start_text)
    context.bot.send_message(
        chat_id=update.effective_chat.id, text=f"{start_text}")  # update.message.from_user.full_name)
# from typing import TypeVar
#
# Cls = TypeVar('Cls')
# def copy_class(cls: Cls) -> Cls:
#     copy_cls = type(f'{cls.__name__}Copy', cls.__bases__, dict(cls.__dict__))
#     for name, attr in cls.__dict__.items():
#         try:
#             hash(attr)
#         except TypeError:
#             # Assume lack of __hash__ implies mutability. This is NOT
#             # a bullet proof assumption but good in many cases.
#             setattr(copy_cls, name, deepcopy(attr))
#     return copy_cls

initial_state = [0,0,0,0]
def init(update, context):
    global d
    global t
    global f
    global sw
    global initial_state
    d = Deck()  # 產生一組鋪克牌
    t = Tableau([d.deal_cards(x) for x in range(1, 8)])  # 先pop出牌面上的牌
    f = Foundation()
    sw = StockWaste(d.deal_cards(24))  # 再pop出24張去stock
    initial_state[0] = copy.deepcopy(d)
    initial_state[1] = copy.deepcopy(t)
    initial_state[2] = copy.deepcopy(f)
    initial_state[3] = copy.deepcopy(sw)


    start_text = printTable(t, f, sw)
    #print('start_text',start_text)
    context.bot.send_message(
        chat_id=update.effective_chat.id, text=f"{start_text}")#update.message.from_user.full_name)

def button(update, context):
    context.bot.send_message(
        chat_id=update.effective_chat.id, text='功能按鈕', reply_markup=InlineKeyboardMarkup([[
            InlineKeyboardButton('翻牌', callback_data='a'),
            InlineKeyboardButton('重新開始', callback_data='b')
        ], ])#[InlineKeyboardButton('功能 C', callback_data='c')]
    )

def bye(update, context):
    context.bot.send_message(
        chat_id=update.effective_chat.id, text="bye {0}!".format(update.message.from_user.full_name))
#
# def repeat(update, context):
#     context.bot.send_message(
#         chat_id=update.effective_chat.id, text=update.message.text * 3)
#
# def add(update, context):
#     sentence = update.message.text[5:]
#     update.message.reply_text('已加入：' + sentence)
def move(update, context):
    move_cmd = update.message.text.split()
    #print(move_cmd)
    # from_where = int(update.message.text[6:7])
    # which = int(update.message.text[8:9])
    # to_where = int(update.message.text[10:11])
    try:
        from_where = int(move_cmd[1])
        which = int(move_cmd[2])
        to_where = int(move_cmd[3])
        print(from_where,which,to_where)
        # if 6 and which and to_where:
        #     print('hiii')
        if from_where == 4 and which == 0 and to_where<4 and to_where>=0:#wf
            if f.addCard_row(sw.getWaste(),row = to_where):
                sw.pop_waste_card()
                start_text = printTable(t, f, sw)
                # print('start_text',start_text)
                context.bot.send_message(
                    chat_id=update.effective_chat.id, text=f"{start_text}")
            else:
                context.bot.send_message(
                    chat_id=update.effective_chat.id, text="Error! No card could be moved from the Waste to the Foundation.")
        elif from_where == 4 and which == 0 and to_where>=5 and to_where<=11:#wt
            col = to_where - 5
            if t.waste_to_tableau(sw, col):
                start_text = printTable(t, f, sw)
                context.bot.send_message(
                    chat_id=update.effective_chat.id, text=f"{start_text}")
            else:
                context.bot.send_message(
                    chat_id=update.effective_chat.id,
                    text="Error! No card could be moved from the Waste to the Tableau column.")
        elif from_where >=5 and from_where <=11 and which == 0 and to_where<4 and to_where>=0:  # tf
            col = from_where - 5
            if t.tableau_to_foundation(f, col,row = to_where):
                start_text = printTable(t, f, sw)
                context.bot.send_message(
                    chat_id=update.effective_chat.id, text=f"{start_text}")
            else:
                context.bot.send_message(
                    chat_id=update.effective_chat.id,
                    text="Error! No card could be moved from the Tableau column to the Foundation.")
        elif from_where >=5 and from_where <=11 and  to_where>=5 and to_where<=11:#tt
            c1, c2 = from_where-5 , to_where - 5
            print(c1,c2)
            if t.tableau_to_tableau(c1, c2,which):
                start_text = printTable(t, f, sw)
                context.bot.send_message(
                    chat_id=update.effective_chat.id, text=f"{start_text}")
            else:
                context.bot.send_message(
                    chat_id=update.effective_chat.id,
                    text="Error! No card could be moved from that Tableau column.")
        else:
            context.bot.send_message(
                chat_id=update.effective_chat.id,
                text="Sorry, that is not a valid command.")
        if f.gameWon():
            context.bot.send_message(
                chat_id=update.effective_chat.id,
                text="Congratulations! You've won!")
    except:
        context.bot.send_message(
            chat_id=update.effective_chat.id,
            text="Sorry, No card could be moved.")

        #update.message.reply_text('已加入：' + str(from_where)+str(which)+str(to_where))
# d = Deck()#產生一組鋪克牌
# t = Tableau([d.deal_cards(x) for x in range(1,8)])#先pop出牌面上的牌
# f = Foundation()
# sw = StockWaste(d.deal_cards(24))#再pop出24張去stock
#

# print("\n" + BREAK_STRING)
# print("Welcome to Danny's Solitaire!\n")
# printValidCommands()
# printTable(t, f, sw)

start_handler = CommandHandler('start', init)
dispatcher.add_handler(start_handler)

button_handler = CommandHandler('button', button)
dispatcher.add_handler(button_handler)
dispatcher.add_handler(CallbackQueryHandler(func))

end_handler = CommandHandler('end', bye)
dispatcher.add_handler(end_handler)

print_handler = CommandHandler('print', print_now)
dispatcher.add_handler(print_handler)

# add_handler = CommandHandler('add',add)
# dispatcher.add_handler(add_handler)

move_handler = CommandHandler('move',move)
dispatcher.add_handler(move_handler)
#
# repeat_handler = MessageHandler(Filters.text & (~Filters.command), repeat)#文字and非指令
# dispatcher.add_handler(repeat_handler)


updater.start_polling()
