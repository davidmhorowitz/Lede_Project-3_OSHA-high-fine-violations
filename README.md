# Lede_Project-3_OSHA-high-fine-violations
README:
This breakdown includes a minority of the code used for the project but goes over the biggest parts of what I did; plenty more, not explained here, went into the project:

I perused OSHA’s datasets and came across a map and a table of high-fine penalties handed out to employers — that is, penalties with fines of $40,000 or more: https://www.osha.gov/enforcement/toppenalties 

I partly chose to focus on *high penalties* because I couldn’t easily find data on *all* penalties — but I do believe that this works as a story angle because it only makes sense that higher-fine penalties are also far more likely to be more *serious* penalties. For a longer project, I’d likely reach out to OSHS to confirm it and get a statement on the significance of these higher-fine penalties.

I originally hoped to use the data to map the *restaurants* with the most penalties by using a “get” request for each penalty to pull up more data on individual penalties. I was also hoping to find how much being unionized correlated with a restaurant having fewer high-fine penalties.

However, when I attempted a “get” request — with the plan to use BeautifulSoup to scrape — I got an “access denied” response.

But I did find *one* undocumented API endpoint. It wasn’t as much as I wanted — and did not allow me to click into an individual high-fine penalty and isolate restaurants or retrieve data on whether an employer was unionized — but I pivoted and drew new conclusions.

I wanted to know which employers are doing the worst and how bad has it gotten — and are these high-fine penalties actually improving the situation? I planned to make a bar chart and a line chart (well, it ended up being a set of three line charts) to answer this.

Using “value_counts” in pandas quickly showed that Dollar Tree, the USPS and Dollar General topped the list, accruing the most high-penalty violations. 

I noticed that the same employers were sometimes listed under different names, so for each of the top 5 employers, I normalized the names with “.title()” and merged employer names:

employers_list_fixed = []

for cell in df['employer']:
    if "Dollar Tree" in cell.strip().title() or "Family Dollar" in cell.strip().title():
        cell_dt = cell.replace(cell, "Dollar Tree/Family Dollar (similar names merged)")
        employers_list_fixed.append(cell_dt)
    elif "Dollar General" in cell.strip().title() or "Dolgencorp" in cell.strip().title():
        cell_dg = cell.replace(cell, "Dollar General (similar names merged)")
        employers_list_fixed.append(cell_dg)
    elif "Postal Service" in cell.strip().title():
        cell_ps = cell.replace(cell, "U.S. Postal Service (similar names merged)")
        employers_list_fixed.append(cell_ps)
    elif "Target" in cell.strip().title():
        cell_tgt = cell.replace(cell, "Target (similar names merged)")
        employers_list_fixed.append(cell_tgt)
    elif "Kaiser" in cell.strip().title():
        cell_ksr = cell.replace(cell, "Kaiser (similar names merged)")
        employers_list_fixed.append(cell_ksr)
    else:
        employers_list_fixed.append(cell)

The results I got from this and “value_counts()” made the bar chart easy work:

df['employers_fixed'].value_counts().head(10)

df_bargraph = pd.DataFrame({
    'Count': df['employers_fixed'].value_counts().head(10)
}).reset_index()

But making a project that is more guaranteed to be unique requires a lot more.

The Occupational Safety & Health Administration has published numerous releases about the three employers violating safety and health regulations. Moreover, so many news outlets have published about Dollar Tree and Dollar General falling short — so I read through several of the top articles to figure out what was lacking. Weirdly, I couldn’t easily find one that showed how many violations each of the companies accrued. It’s easy enough to crunch that I thought I need to find something else, especially in the event that an article exists that has already done that.

The latter question — whether things are improving or not — would require a lot more work. And, given that the top articles on Dollar Tree and Dollar General don’t attempt to do it, I doubt it’s been done.

So, making a sound argument about this would become my focus:

I knew I would eventually look at the penalties by year. I used regex and a Python function to get this:

def get_year(date):
    pattern = r"^\d\d/\d\d/(\d{4})"
    replacement = r"\1"
    return int(re.sub(pattern, replacement, date))
print(get_year('12/30/2023'))

df['issuance_date'].str.replace(r"^\d\d/\d\d/(\d{4})", r"\1", regex=True)

For each employer and each year, I looked at both the amount accrued in the high penalties per year as well as the *number* of penalties per year: 

E.g.,

This is the code for one employer owed in high-fine penalties *in dollars* for one year: df['initial_penalty_cleaned'][df.Year == 2023][df.employers_fixed == 'Dollar Tree/Family Dollar (similar names merged)'].sum()

And this is the code for one employer’s *number* of high-fine penalties for one year:

len(df.loc[((df["employers_fixed"] == 'Dollar Tree/Family Dollar (similar names merged)') & (df["Year"] == 2023))])

I decided that since Dollar Tree, Dollar General and Target are the top three private employers with the most high-fine OSHA violations *and* direct competitors, I would compare the three. Target could be a baseline that the dollar store companies could be compared to.

I looked at Dollar Tree. It did not improve despite accruing a substantial increase in high fines in 2022. In fact, I found that it is projected to do significantly *worse* this year. 

When I got to Dollar General’s number of high-fine penalties per year, I was like, “Holy shit”:

2015: 7
2016: 11
2017: 7
2018: 5
2019: 2
2020: 6
2021: 9
2022: 29
2023: 48

So, OSHA’s highest penalties *clearly* aren’t stopping the two greatest violators of workers’ health and safety laws in the U.S. That's the angle.

Target, in comparison, seemed relatively stable. But I wanted a more sound argument: To ensure it’s a fair comparison between the dollar store companies and Target, I had to look at each’s employee counts.

I retrieved the annual 10-K reports filed with the SEC for each company, used “cmd+f” and typed in “full-” to quickly get the number of employees for each company each year since 2016.

I chose 2016 for two reasons: Dollar Tree did not have an SEC report before 2016, and Target still had stores in Canada in early 2015 (they closed before 2016).

Dollar General appeared to not have stores outside of the U.S. Dollar Tree has stores in Canada, but they appear a tiny minority, so I used a multiplier. Here’s an explanation of the multiplier from my Jupyter Notebook file:

Scrapehero indicates that there are 233 Dollar Trees in Canada. (I can't find any indication online of
Family Dollar — which was acquired by Dollar Tree — existing in Canada.) Scrapehero seems very likely to be overall accurate because its count of
Dollar Trees and Family Dollars, in total, are very close to the figures in the company's 10-K reports:

Scrapehero states: 8349 Family Dollars + 7927 U.S. Dollar Trees + 233 Canada Dollar Trees = 16509. 
Dollar Tree in the SEC report counts 16340.

Assuming this is roughly accurate, we can say that Dollar Tree/Family Dollar's Canada stores make up
only 233 / 16509 of the company's locations, or 1.4%. 

It's not exact, but this is such a small figure that I felt it negated the need for exact precision, and we can likely 
extrapolate to figure out the number of employees in the U.S. 
This means that we’re going to multiply the number of employees by 0.986 . 
Ideally, we would get Scrapehero's figures for Canada stores per year, but the figure is so relatively small
that I think it's okay if, for this project, I use the 0.986 multiplier.

Another possible weakness is that I didn’t account for the ratio of full-time employees to part-time employees; however, given that these businesses are direct competitors in the same industry, I figure that it’s okay, overall, to work with the total number of employees, especially given that it’s just the denominator to make my comparison among companies — “dollars owed per employee” — more accurate.

Anyway, I eventually got the number of dollars per employee. For 2023, I projected from the end of July (given that I got the data on July 24) by multiplying the OSHA numbers by 12 and dividing by 7. I used the yearly dollar amounts owed for Dollar General, Dollar Tree and Target to create my line charts — which show that Dollar General and Dollar Tree blow Target out of the water when it comes to violations per employee. And, because I also crunched data on the *number* of penalties, I know that it aligns fairly well with the dollars owed per employee, meaning that outliers aren’t playing a role in Dollar General and Dollar Tree’s great number of violations per employee.

Here are weaknesses I didn’t mention:
-I think that I should include a chart for the *number* of penalties per employee.
-The SEC reports, which I pull employee count data from, come out early in the year, yet I compare them to end-of-year OSHA violation counts. I don’t think it matters too much, given that the exact number of employees is less important than the number of OSHA violations, and that the change in the number of employees per year never seems that substantial — especially when compared to how much the OSHA violations changed per year. Also, unless I’m mistaken, my doing the same thing for each company, I think, somewhat negates the relevance of this.
-If I were to spend more time making this more complete, I would use browser automation to figure out the addresses with the most violations each year. I would attempt to figure out which penalties are most common for each company. And, I would go to these stores for anecdotes from employees where things went wrong.
