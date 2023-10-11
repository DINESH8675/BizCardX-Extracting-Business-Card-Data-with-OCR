import pandas as pd
import streamlit as st
from streamlit_option_menu import option_menu
import easyocr
from PIL import Image
import cv2
import os
import matplotlib.pyplot as plt
import re
import psycopg2


icon = Image.open("/Users/vpro/Downloads/Iconocr.png")
st.set_page_config(page_title="BizCardX: Extracting Business Card Data with OCR ",
                   page_icon=icon,
                   layout="wide",
                   initial_sidebar_state="auto")
st.markdown("<h1 style='text-align: center; color: violet;'>BizCardX: Extracting Business Card Data with OCR</h1>",
            unsafe_allow_html=True)
SELECT = option_menu(
    menu_title = None,
    options = ["Upload","Modify","Delete"],
    icons =["house","cloud-upload","pencil-square"],
    default_index=0,
    orientation="horizontal",
    styles={"container": {"padding": "0!important", "background-color": "white"},
        "icon": {"color": "violet", "font-size": "15px"},
        "nav-link": {"font-size": "15px", "text-align": "centre", "margin": "0px", "--hover-color": "#F0F2F6"},
        "nav-link-selected": {"background-color": "#5DADE2"}})

#connecting to SQL database

kumar =psycopg2.connect(host='localhost', user='postgres', password='DINESHKUMAR', port=5432,database='bizcard')
dinesh = kumar.cursor()
dinesh.execute(f"""create table if not exists card_data(
                                        company_name TEXT,
                                        card_holder TEXT,
                                        designation TEXT,
                                        mobile_number VARCHAR(50),
                                        email TEXT,
                                        website TEXT,
                                        area TEXT,
                                        city TEXT,
                                        state TEXT,
                                        pin_code VARCHAR(10))""")
kumar.commit()
# check data in already exist in data base
if SELECT == "Upload":
    Downloads = st.file_uploader("upload here", label_visibility="collapsed", type=["png", "jpeg", "jpg"])
    reader = easyocr.Reader(['en'], gpu=True,verbose=False)
    if Downloads is not None:

        def save_card(Downloads):
            with open(os.path.join( Downloads.name), "wb") as f:
                f.write(Downloads.getbuffer())


        save_card(Downloads)

        def image_preview(image, res):
            for (bbox, text, prob) in res:
                # unpack the bounding box
                (tl, tr, br, bl) = bbox
                tl = (int(tl[0]), int(tl[1]))
                tr = (int(tr[0]), int(tr[1]))
                br = (int(br[0]), int(br[1]))
                bl = (int(bl[0]), int(bl[1]))
                cv2.rectangle(image, tl, br, (0, 255, 0), 2)
                cv2.putText(image, text, (tl[0], tl[1] - 10),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.7, (255, 0, 0), 2)
            plt.rcParams['figure.figsize'] = (15, 15)
            plt.axis('off')
            plt.imshow(image)


        # DISPLAYING THE UPLOADED CARD
        col1, col2 = st.columns(2, gap="large")
        with col1:
            st.markdown("#     ")
            st.markdown("#     ")
            st.markdown("### :blue[uploaded card] ")
            st.image(Downloads)
        # DISPLAYING THE CARD WITH HIGHLIGHTS
        with col2:
            st.markdown("#  ")
            st.markdown("#  ")
            with st.spinner(":green[Please wait processing image...]"):
                st.set_option('deprecation.showPyplotGlobalUse', False)
                saved_img = os.getcwd() +  "/" + Downloads.name
                image = cv2.imread(saved_img)
                res = reader.readtext(saved_img)
                st.markdown("### :red[Image Processed and Data Extracted]")
                st.pyplot(image_preview(image, res))

                # easy OCR
        saved_img = os.getcwd()  + "//" + Downloads.name
        result = reader.readtext(saved_img,  detail=0, paragraph=False)
        data = {"company_name": [],
                "card_holder": [],
                "designation": [],
                "mobile_number": [],
                "email": [],
                "website": [],
                "area": [],
                "city": [],
                "state": [],
                "pin_code": []
                }


        def get_data(result):
            for j, i in enumerate(result):

                # To get WEBSITE_URL
                if "www " in i.lower() or "www." in i.lower():
                    data["website"].append(i)
                elif "WWW" in i:
                    data["website"] = res[4] + "." + res[5]

                # To get EMAIL ID
                elif "@" in i:
                    data["email"].append(i)

                # To get MOBILE NUMBER
                elif "-" in i:
                    data["mobile_number"].append(i)
                    if len(data["mobile_number"]) == 2:
                        data["mobile_number"] = " & ".join(data["mobile_number"])

                # To get COMPANY NAME
                
                elif j == len(result) -1:
                    data["company_name"].append(''.join(i))

                # To get CARD HOLDER NAME
                elif j == 0:
                    data["card_holder"].append(i)

                # To get DESIGNATION
                elif j == 1:
                    data["designation"].append(i)

                # To get AREA
                if re.findall('^[0-9].+, [a-zA-Z]+', i):
                    data["area"].append(i.split(',')[0])
                elif re.findall('[0-9] [a-zA-Z]+', i):
                    data["area"].append(i)

                # To get CITY NAME
                match1 = re.findall('.+St , ([a-zA-Z]+).+', i)
                match2 = re.findall('.+St,, ([a-zA-Z]+).+', i)
                match3 = re.findall('^[E].*', i)
                if match1:
                    data["city"].append(match1[0])
                elif match2:
                    data["city"].append(match2[0])
                elif match3:
                    data["city"].append(match3[0])

                # To get STATE
                state_match = re.findall('[a-zA-Z]{9} +[0-9]', i)
                if state_match:
                    data["state"].append(i[:9])
                elif re.findall('^[0-9].+, ([a-zA-Z]+);', i):
                    data["state"].append(i.split()[-1])
                if len(data["state"]) == 2:
                    data["state"].pop(0)

                # To get PINCODE
                if len(i) >= 6 and i.isdigit():
                    data["pin_code"].append(i)
                elif re.findall('[a-zA-Z]{9} +[0-9]', i):
                    data["pin_code"].append(i[10:])


        get_data(result)
        def create_df(data):
            df = pd.DataFrame(data)
            return df


        df = create_df(data)
        st.success("### Data Extracted Successfully")
        st.write(df)
        
        if st.button("Upload to Database"):
            for i, row in df.iterrows():
                
                sql = """INSERT INTO card_data(company_name,card_holder,designation,mobile_number,email,website,area,city,state,pin_code)
                         VALUES (%s,%s,%s,%s,%s,%s,%s,%s,%s,%s)"""
                dinesh.execute(sql, tuple(row))
               
                kumar.commit()
                st.success("#### Uploaded to database successfully!")

        if st.button(":blue[View updated data]"):
            dinesh.execute("select company_name,card_holder,designation,mobile_number,email,website,area,city,state,pin_code from card_data")
            updated_df = pd.DataFrame(dinesh.fetchall(),
                                          columns=["Company_Name", "Card_Holder", "Designation", "Mobile_Number",
                                                   "Email",
                                                   "Website", "Area", "City", "State", "Pin_Code"])
            st.write(updated_df)

# MODIFY MENU


if  SELECT == "Modify":
        st.markdown(":blue[Alter the data here]")

        try:
            dinesh.execute("SELECT card_holder FROM card_data")
            result = dinesh.fetchall()
            business_cards = {}
            for row in result:
                business_cards[row[0]] = row[0]
            options = ["None"] + list(business_cards.keys())
            selected_card = st.selectbox("**Select a card**", options)
            if selected_card == "None":
                st.write("No card selected.")
            else:
                st.markdown("#### update or modify any data below")
                dinesh.execute(
                "select company_name,card_holder,designation,mobile_number,email,website,area,city,state,pin_code from card_data WHERE card_holder=%s",
                (selected_card,))
                result = dinesh.fetchone()

                # DISPLAYING ALL THE INFORMATIONS
                company_name = st.text_input("Company_Name", result[0])
                card_holder = st.text_input("Card_Holder", result[1])
                designation = st.text_input("Designation", result[2])
                mobile_number = st.text_input("Mobile_Number", result[3])
                email = st.text_input("Email", result[4])
                website = st.text_input("Website", result[5])
                area = st.text_input("Area", result[6])
                city = st.text_input("City", result[7])
                state = st.text_input("State", result[8])
                pin_code = st.text_input("Pin_Code", result[9])


                if st.button(":blue[Commit changes to DB]"):


                   # Update the information for the selected business card in the database
                    dinesh.execute("""UPDATE card_data SET company_name=%s,card_holder=%s,designation=%s,mobile_number=%s,email=%s,website=%s,area=%s,city=%s,state=%s,pin_code=%s
                                    WHERE card_holder=%s""", (company_name, card_holder, designation, mobile_number, email, website, area, city, state, pin_code,
                    selected_card))
                    kumar.commit()
                    st.success("Information updated in database successfully.")

            if st.button(":blue[View updated data]"):
                dinesh.execute(
                    "select company_name,card_holder,designation,mobile_number,email,website,area,city,state,pin_code from card_data")
                updated_df = pd.DataFrame(dinesh.fetchall(),
                                          columns=["Company_Name", "Card_Holder", "Designation", "Mobile_Number",
                                                   "Email",
                                                   "Website", "Area", "City", "State", "Pin_Code"])
                st.write(updated_df)

        except:
            st.warning("There is no data available in the database")

if SELECT == "Delete":
        st.subheader(":blue[Delete the data]")
        try:
            dinesh.execute("SELECT card_holder FROM card_data")
            result = dinesh.fetchall()
            business_cards = {}
            for row in result:
                business_cards[row[0]] = row[0]
            options = ["None"] + list(business_cards.keys())
            selected_card = st.selectbox("**Select a card**", options)
            if selected_card == "None":
                st.write("No card selected.")
            else:
                st.write(f"### You have selected :violet[**{selected_card}**] card to delete")
                st.write("#### Proceed to delete this card?")
                if st.button("Yes Delete Business Card"):
                    dinesh.execute(f"DELETE FROM card_data WHERE card_holder='{selected_card}'")
                    kumar.commit()
                    st.success("Business card information deleted from database.")

            if st.button(":blue[View updated data]"):
                dinesh.execute(
                    "select company_name,card_holder,designation,mobile_number,email,website,area,city,state,pin_code from card_data")
                updated_df = pd.DataFrame(dinesh.fetchall(),
                                          columns=["Company_Name", "Card_Holder", "Designation", "Mobile_Number",
                                                   "Email",
                                                   "Website", "Area", "City", "State", "Pin_Code"])
                st.write(updated_df)

        except:
            st.warning("There is no data available in the database")
     
