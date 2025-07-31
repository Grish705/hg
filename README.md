# hg

from pprint import pprint
import csv
import re


with open("phonebook_raw.csv", encoding="utf-8") as f:
    rows = csv.reader(f, delimiter=",")
    contacts_list = list(rows)


for contact in contacts_list[1:]:
    name_parts = contact[:3]
    full_name = " ".join(name_parts).split()

    contact[0] = full_name[0]
    contact[1] = full_name[1] if len(full_name) > 1 else ''
    contact[2] = full_name[2] if len(full_name) > 2 else ''


phone_pattern = re.compile(
    r"(\+7|8)?\s*$?(\d{3})$?[\s-]*(\d{3})[\s-]*(\d{2})[\s-]*(\d{2})(?:\s*(доб.)?\s*(\d+))?"
)

for contact in contacts_list[1:]:
    phone = contact[5]
    if phone:
        match = phone_pattern.search(phone)
        if match:
            country_code = "+7"
            area_code = match.group(2)
            first_part = match.group(3)
            second_part = match.group(4)
            third_part = match.group(5)
            extension_label = match.group(6)
            extension_number = match.group(7)

            formatted_phone = f"{country_code}({area_code}){first_part}-{second_part}-{third_part}"
            if extension_label and extension_number:
                formatted_phone += f" {extension_label}.{extension_number}"
            contact[5] = formatted_phone


contacts_dict = {}
for contact in contacts_list[1:]:
    key = (contact[0], contact[1])
    if key not in contacts_dict:
        contacts_dict[key] = contact
    else:
        existing_contact = contacts_dict[key]
        for i in range(len(contact)):
            if not existing_contact[i] and contact[i]:
                existing_contact[i] = contact[i]


contacts_list_cleaned = [contacts_list[0]] + list(contacts_dict.values())


pprint(contacts_list_cleaned)


with open("phonebook.csv", "w", encoding="utf-8", newline='') as f:
    datawriter = csv.writer(f, delimiter=',')
    datawriter.writerows(contacts_list_cleaned)
