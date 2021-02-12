---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---

# Item Dashboard report

## How to create the item dashboard report

The SQL for this report is kind of a lot:

<mark>(Warning - if you copy and paste this report directly into your system - it will not work until you create the accessory reports and edit this SQL with the correct report numbers)</mark>

{% highlight SQL linenos %}

SELECT
  CONCAT_WS("<br />",
    '<h3 style="color: white; background-color: #829356; text-align: center;">This item is currently in the catalog</h3>',
    Concat("Home library: ", items.homebranch),
    Concat("Current library: ", items.holdingbranch),
    Concat("Permanent location: ", permanent_locss.lib),
    Concat("Current location: ", locss.lib),
    Concat("Item type: ", itemtypess.description),
    Concat("Collection code: ", ccodes.lib),
    Concat("Call number: ", items.itemcallnumber),
    Concat("Author: ", biblio.author),
    Concat("Title: ",
      Concat_Ws(' ',
        biblio.title, ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="h"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="b"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="n"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="p"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="c"]')
      )
    ),
    Concat("Item barcode: ", items.barcode),
    Concat("Public notes: ", items.itemnotes),
    Concat("Non-public notes: ", items.itemnotes_nonpublic),
    Concat("<br />Checkouts: ", items.issues),
    Concat("Renewals: ", items.renewals),
    Concat("Date added: ", items.dateaccessioned),
    Concat("Last borrowed: ", items.datelastborrowed),
    Concat("Last seen: ", items.datelastseen),
    Concat("Item record last modified: ", items.timestamp),
    Concat("Due date: ", issuesi.date_due),
    Concat("Not for loan status: ", notforloans.lib),
    Concat("Damaged: ", Concat(damageds.lib, ' ', items.damaged_on)),
    Concat("Lost: ", Concat(losts.lib, ' ', items.itemlost_on)),
    Concat("Withdrawn: ", Concat(withdrawns.lib, ' ', items.withdrawn_on)),
    Concat(
      "In transit from: ",
      If(
        transfersi.frombranch IS NULL,
        "-",
        Concat(transfersi.frombranch, " to ", transfersi.tobranch, " since ", transfersi.datesent)
      )
    ),
    Concat(
      "<br />Link to borrower: ",
      If(
        issuesi.date_due IS NULL,
        "-",
        Concat(
          "<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=",  
          issuesi.borrowernumber,
          "' target='_blank'>go to the borrower's account</a>"
        )
      )
    ),
    Concat(
      "Link to title: ",
      Concat(
        "<a href='/cgi-bin/koha/catalogue/detail.pl?biblionumber=",
        biblio.biblionumber,
        "' target='_blank'>go to the bibliographic record</a>"
      )
    ),
    Concat(
      "Link to item: ",
      Concat(
        "<a href='/cgi-bin/koha/catalogue/moredetail.pl?itemnumber=",
        items.itemnumber,
        "&biblionumber=",
        biblio.biblionumber,
        "' target='_blank'>go to the item record</a>"
      )
    ),
    Concat(
      "<br />Item circ history: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2785&phase=Run+this+report&param_name=Enter+item+barcode+number&sql_params=",
        items.barcode,
        "' target='_blank'>see item circ history</a>"
      )
    ),
    Concat(
      "Item action log history: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3342&phase=Run+this+report&param_name=Enter+item+number&sql_params=",
        items.itemnumber,
        "' target='_blank'>see action log history</a>"
      )
    ),     
    Concat(
      "Item in transit history: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2784&phase=Run+this+report&sql_params=",
        items.barcode,
        "' target='_blank'>see item transit history</a>"
      )
    ),
    Concat(
      "Request history on this title: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=",
        biblio.biblionumber,
        "&sql_params=%25' target='_blank'>see title's request history</a>"
      )
    ),
    Concat(
      "Request history on this item: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=",
        items.barcode,
        "' target='_blank'>see item's request history</a>"
      )
    ),
    '<br /><h3 style="color: white; background-color: #829356; text-align: center;">This item is currently in the catalog</h3>'
  ) AS INFO
FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
     itemtypes.itemtype,
     itemtypes.description
   FROM
     itemtypes) itemtypess ON itemtypess.itemtype = items.itype LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn JOIN
  biblio_metadata ON biblio_metadata.biblionumber = biblio.biblionumber
  LEFT JOIN
  (SELECT
     issues.itemnumber,
     issues.date_due,
     issues.borrowernumber
   FROM
     issues) issuesi ON issuesi.itemnumber = items.itemnumber LEFT JOIN
  (SELECT
     branchtransfers.itemnumber,
     branchtransfers.frombranch,
     branchtransfers.datesent,
     branchtransfers.tobranch,
     branchtransfers.datearrived
   FROM
     branchtransfers
   WHERE
     branchtransfers.datearrived IS NULL) transfersi ON transfersi.itemnumber =
      items.itemnumber
WHERE
  items.barcode LIKE Concat('%', <<Enter item barcode number>>, '%')
GROUP BY
  items.itemnumber,
  biblio.biblionumber
UNION
SELECT
  CONCAT_WS("<br />",
    '<h2 style="color: white; background-color: #AD2A1A; text-align: center;">This item has been deleted</h2>',
    Concat('Home library: ', deleteditems.homebranch),
    Concat('Current library: ', deleteditems.holdingbranch),
    Concat('Permanent location: ', deleteditems.permanent_location),
    Concat('Current location: ', deleteditems.location),
    Concat('Item type: ', deleteditems.itype),
    Concat('Collection code: ', ccodes.lib),
    Concat('Call#: ', deleteditems.itemcallnumber),
    Concat('Author: ', Coalesce(biblio.author, deletedbiblio.author)),
    Concat('Title: ', Coalesce(biblio.title, deletedbiblio.title)),
    Concat('Item barcode: ', deleteditems.barcode),
    Concat('Replacement price: ', deleteditems.replacementprice),
    Concat('Item id number: ', deleteditems.itemnumber),
    Concat(
      "<br />Damaged status: ",
      If(
        deleteditems.damaged = 0,
        "-",
        If(
          deleteditems.damaged IS NULL,
          "-",
          damageds.lib
        )
      )
    ),
    Concat(
      "Lost status: ",
      If(
        deleteditems.itemlost = 0,
        "-",
        If(
          deleteditems.itemlost IS NULL,
          "-",
          Concat(losts.lib, " on ", deleteditems.itemlost_on)
        )
      )
    ),
    Concat(
      "Withdrawn status: ",
      If(
        deleteditems.withdrawn = 0,
        "-",
        If(
          deleteditems.withdrawn IS NULL,
          "- ",
          Concat(withdrawns.lib, " on ", deleteditems.withdrawn_on)
        )
      )
    ),
    Concat(
      ": ",
      If(
        biblio.biblionumber IS NULL,
        "<br />-- Bibliographic record has been deleted --",
        Concat(
          "<br /><a href='/cgi-bin/koha/catalogue/detail.pl?biblionumber=",
          biblio.biblionumber,
          "' target='_blank'>Go to the bibliographic record</a>"
        )
      )
    ),
    Concat(
      "<br /><a href='/cgi-bin/koha/reports/guided_reports.pl?phase=Run+this+report&reports=3009&sql_params=",
      Replace(
        Replace(
          Replace(
            Replace(
              Replace(
                Replace(
                  Replace(
                    deleteditems.barcode,
                    Char(43),
                    "%2B"),
                  Char(47),
                  "%2F"),
                Char(32),
                "%20"),
              Char(45),
              "%2D"),
            Char(36),
            "%24"),
          Char(37),
          "%25"),
        Char(46),
        "%2E"
      ),
      "&limit=50' target='_blank'>Search payment and fee notes and descriptions for this item barcode number</a>"
    ),
    '<br /><h2 style="color: white; background-color: #AD2A1A; text-align: center;">This item was deleted from the catalog<br />within the past 13 months</h2>'
  ) AS INFO
FROM
  deleteditems LEFT JOIN
  biblio ON deleteditems.biblionumber = biblio.biblionumber LEFT JOIN
  deletedbiblio ON deletedbiblio.biblionumber = deleteditems.biblionumber
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = deleteditems.permanent_location
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      deleteditems.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = deleteditems.itype
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      deleteditems.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = deleteditems.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = deleteditems.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      deleteditems.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = deleteditems.withdrawn
WHERE
  deleteditems.barcode LIKE Concat('%', <<Enter item barcode number>>, '%')
GROUP BY
  deleteditems.itemnumber,
  biblio.biblionumber

{% endhighlight %}




<mark>(Warning - if you copy and paste this report directly into your system - it will not work until you create the accessory reports and edit this SQL with the correct report numbers)</mark>

Yes.  I know.  It's a big report.  It's got a lot going on in it.

Here's what it does:

If you enter an item barcode number and that barcode number is in items.barcode or deleteditems.barcode, you will get a result that lists pertinent information about the item in a format that's easy to cut-and-paste into a document or an e-mail.

If the item has not been deleted, you'll get links to the item record, the bibliographic record, and 5 other reports.  A report for circulation history, a report for action log history, a report for the item's in-transit history, and reports for the title's request history and the item's request history.

If the item has been deleted, you'll get a link to the bibliographic record (unless the biblio has also been deleted) and a link to a report that will show you any accountlines where the item barcode number appears in the accountline description or note fields.

Here's why it does all of this:

I manage a Koha that is shared by 51 libraries and uses a courier service to move materials across 7400 square miles of north east Kansas.  Of these 51 libraries, the smallest serves a community of less than 200 and the largest serves a community of over 35,000.  The number of staff as well as their experience and education varies from library to library.  It is not uncommon for me to receive phone calls or e-mails saying, "I have an item with barcode number X.  I need to know more about it."

I can use Koha to find out a ton of things about an item.  I can tell you where it's been and when it was there.  I can tell you if it's still active in the system or if it's been deleted within the last 13 months (we run a script that purges the deleted items table of records that have been in deleteditems for more than 13 months).  I can tell you its circulation history, its requset history, and other things with just the barcode number.  But I have to go to a bunch of different places to find these things out.

I wrote this report to be a one-stop-shop for an item barcode number search.  If I get an e-mail saying "I have this barcode number and when I scan it I get __ happens.  Why does __ happen."

With this report, I can scan the barcode number once and if the barcode number is still active or has been deleted within the last 13 months, I get a result.  I don't have to run separate reports for deleted and non-deleted items.

If the item is still active, I can easily copy and paste item information into an e-mail, go to more specific item data, or run reports about circulation history, action log history, in transit history, or the item's request history.  And if it's been deleted within the last 13 months, I get a result that I can easily copy and past into an e-mail and a link to references in the accountlines notes and descriptions where the barcode number can be found.  And if it was deleted more than 13 months ago, I get no result.  In short, this report puts 90% of the things I need for answering questions about items into 1 spot.

----------

So, in order to help others understand how this report was built, I'm going to walk through all of the steps I took to build it, one step at a time.

## Step 1

I started with a report that looks at basic item information from the items and biblio table.

{% highlight SQL linenos %}

SELECT
  items.homebranch,
  items.holdingbranch,
  items.permanent_location,
  items.location,
  items.itype,
  items.ccode,
  items.itemcallnumber,
  biblio.author,
  biblio.title,
  items.barcode,
  items.itemnotes,
  items.itemnotes_nonpublic,
  items.issues,
  items.renewals,
  items.dateaccessioned,
  items.datelastborrowed,
  items.datelastseen,
  items.timestamp,
  items.onloan,
  items.notforloan,
  items.damaged,
  items.damaged_on,
  items.itemlost,
  items.itemlost_on,
  items.withdrawn,
  items.withdrawn_on
FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber
WHERE
  items.barcode LIKE CONCAT('%', <<Enter item barcode number>>, '%')
GROUP BY
  items.itemnumber,
  biblio.biblionumber

{% endhighlight %}

This report gets me the basic information I want for an item that is still in the system, but I really want something I can cut and paste into an e-mail that's easy to read and that will make as much sense to a brand new employee who's straight out of high school as to a library director with a Masters degreen and 35 years of experience.  

## Step 2

My next step is to add some sub-queries that will convert some of the codes into descriptions (i.e. instead of item type = "NVIDNEW" staff will item type = "Video (new)"; instead of itemlost = 2, staff will see itemlost = "Lost (more than 45 days overdue)."

If you don't know how to create sub-queries, please check out the video: <a href="https://youtu.be/iRBEvt4nDbU" target="_blank">SQL: Dates and Subqueries</a>

As a first sub-query example, this next sample code includes just a sub-query for getting the collection code description instead of just the collection code authorised value:

{% highlight SQL linenos %}

SELECT
  items.homebranch,
  items.holdingbranch,
  items.permanent_location,
  items.location,
  items.itype,

/* Changes items.ccode to ccodes.lib */

  ccodes.lib AS CCODE,


  items.itemcallnumber,
  biblio.author,
  biblio.title,
  items.barcode,
  items.itemnotes,
  items.itemnotes_nonpublic,
  items.issues,
  items.renewals,
  items.dateaccessioned,
  items.datelastborrowed,
  items.datelastseen,
  items.timestamp,
  items.onloan,
  items.notforloan,
  items.damaged,
  items.damaged_on,
  items.itemlost,
  items.itemlost_on,
  items.withdrawn,
  items.withdrawn_on
FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber

/* Sub-querie called "ccodes" - left joined to items.ccode */

  LEFT JOIN
    (SELECT
        authorised_values.category,
        authorised_values.authorised_value,
        authorised_values.lib
      FROM
        authorised_values
      WHERE
        authorised_values.category = 'ccode') ccodes ON ccodes.authorised_value =
        items.ccode


WHERE
  items.barcode LIKE CONCAT('%', <<Enter item barcode number>>, '%')
GROUP BY
  items.itemnumber,
  biblio.biblionumber

{% endhighlight %}

The two changes involve changing "items.ccode" into "ccodes.lib AS CCODE" and creating the following as a sub-query called "ccodes"

{% highlight SQL linenos %}

SELECT
    authorised_values.category,
    authorised_values.authorised_value,
    authorised_values.lib
  FROM
    authorised_values
  WHERE
    authorised_values.category = 'ccode'

{% endhighlight %}

This sub-query gets a list of all of the collection codes in your system.  By left-joining it to the items table, I'm telling the report to grab the collection code description (if there is one) and use it instead of items.ccode which displays the collection code authorised value.  The description is usually a lot easier to read for inexperienced staff than the authorised value.  This is especially true of the numeric codes used for items.notforloan, items.itemlost, items.damaged, etc.

----------

That sample should get you a good idea of how to use a sub-query, so this next sample includes all of the fields where I want to use a sub-query instead of just a code:

{% highlight SQL linenos %}

SELECT
  items.homebranch,
  items.holdingbranch,

/* Switches source of permanent location, location, and item type from codes to descriptions */

  permanent_locss.lib AS PERM_LOCATION,
  locss.lib AS LOCATION,
  itemtypess.description AS ITYPE,


  ccodes.lib AS CCODE,
  items.itemcallnumber,
  biblio.author,
  biblio.title,
  items.barcode,
  items.itemnotes,
  items.itemnotes_nonpublic,
  items.issues,
  items.renewals,
  items.dateaccessioned,
  items.datelastborrowed,
  items.datelastseen,
  items.timestamp,
  items.onloan,

/* Switches source of damaged, lost and withdrawn from codes to descriptions */

  notforloans.lib,
  damageds.lib AS DAMAGED,
  items.damaged_on,
  losts.lib AS LOST,
  items.itemlost_on,
  withdrawns.lib AS WITHDRAWN,
  items.withdrawn_on


FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber

/* Sub queries to get descriptions for permanent location, location, and item type data */

LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = items.itype


LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode

/* Sub queries to get descriptions for not-for-loan, damaged, lost, and withdrawn data */

LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn


WHERE
  items.barcode LIKE CONCAT('%', <<Enter item barcode number>>, '%')
GROUP BY
  items.itemnumber,
  biblio.biblionumber

{% endhighlight %}

An important thing to remember here is that I want all of these joins to be LEFT JOIN s.  This way if an item doesn't have a "Lost" status or a "Not for loan" status, etc., you'll still get a result.  If you do a simple JOIN, and there is no result in the sub-query, you won't get any result at all.

## Step 3

One of the things that would be nice is if the "Lost" and "Withdrawn" information and the lost_on and withdrawn_on dates appeared together instead of in separate columns.  That's why this next sample concatenates damaged and damaged_on; lost and lost_on; and withdrawn and withdrawn_on into three columns instead of six

If you don't know how to use the "Concat" function, please check out the video: <a href="https://youtu.be/xq9oQ1iP6c0" target="_blank">Concatenating and If Statements in Reports</a>

{% highlight SQL linenos %}

SELECT
  items.homebranch,
  items.holdingbranch,
  permanent_locss.lib AS PERM_LOCATION,
  locss.lib AS LOCATION,
  itemtypess.description AS ITYPE,
  ccodes.lib AS CCODE,
  items.itemcallnumber,
  biblio.author,
  biblio.title,
  items.barcode,
  items.itemnotes,
  items.itemnotes_nonpublic,
  items.issues,
  items.renewals,
  items.dateaccessioned,
  items.datelastborrowed,
  items.datelastseen,
  items.timestamp,
  items.onloan,
  notforloans.lib,

/* Combines status and status dates into single fields */

  CONCAT(damageds.lib, ' ', items.damaged_on) AS DAMAGED,
  CONCAT(losts.lib, ' ', items.itemlost_on) AS LOST,
  CONCAT(withdrawns.lib, ' ', items.withdrawn_on) AS WITHDRAWN


FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = items.itype LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn
WHERE
  items.barcode LIKE CONCAT('%', <<Enter item barcode number>>, '%')
GROUP BY
  items.itemnumber,
  biblio.biblionumber

{% endhighlight %}

This takes the report from 26 columns to 23 columns.  That's still a lot to cut and paste into an e-mail.

## Step 4

I know that if I want this report to be useful for library staff, I'm going to have to add some more title information.  The biblio.title field only includes data from the 245$a in the Marc record.  If I want 245$b, $n, $p, $h, and $c I can add those fields directly from the database if I've got my Koha to Marc mapping set up to do that.  Or I can just extract the data from the biblio_metadata table directly from the Marc records.  Since I started working with Koha before some of this information was able to be mapped, I tend to join to the biblio_metadata table and go that route.  This also uses CONCAT_WS which is mentioned in the video at <a href="https://youtu.be/xq9oQ1iP6c0" target="_blank">Concatenating and If Statements in Reports.</a>

To see a video about extracting data from Marc records in the biblio_metadata table, please check out the video: <a href="https://youtu.be/VrjGxaoRCIw" target="_blank">SQL: ExtractValue.</a>

{% highlight SQL linenos %}

SELECT
  items.homebranch,
  items.holdingbranch,
  permanent_locss.lib AS PERM_LOCATION,
  locss.lib AS LOCATION,
  itemtypess.description AS ITYPE,
  ccodes.lib AS CCODE,
  items.itemcallnumber,
  biblio.author,

/* Adds 245$b, $n, and $p fields to the title */

  Concat_Ws(' ',
    biblio.title,
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="b"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="n"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="p"]')
  ) AS FULL_TITLE,


  items.barcode,
  items.itemnotes,
  items.itemnotes_nonpublic,
  items.issues,
  items.renewals,
  items.dateaccessioned,
  items.datelastborrowed,
  items.datelastseen,
  items.timestamp,
  items.onloan,
  notforloans.lib,
  Concat(damageds.lib, ' ', items.damaged_on) AS DAMAGED,
  Concat(losts.lib, ' ', items.itemlost_on) AS LOST,
  Concat(withdrawns.lib, ' ', items.withdrawn_on) AS WITHDRAWN
FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = items.itype LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn

/* Left joins biblio_metadata to enable extraction of Marc fields */

JOIN
  biblio_metadata ON biblio_metadata.biblionumber = biblio.biblionumber


WHERE
  items.barcode LIKE Concat('%', <<Enter item barcode number>>, '%')
GROUP BY
  items.itemnumber,
  biblio.biblionumber

{% endhighlight %}

## Step 5

At this point I also know that, if the item is checked out, I'm going to want to have a quick way to go to the borrower's record.  Unfortunately there isn't anything in the item record that tells me who an item is checke out to.  In order to get information about who an item is checked out to now, I need to link to the issues table, but I only want a result if the item is actually checked out.  This leads us to our first if/then statement of the report.  It also leads to the first HTML link built into the report.

If you don't know how to create an if/then statement, please check out the video: <a href="https://youtu.be/xq9oQ1iP6c0" target="_blank">Concatenating and If Statements in Reports.</a>

If you don't know how to create a link in a report, please check out the video: <a href="https://youtu.be/71ETEh_cFH0" target="_blank">Links in Reports</a>

I'm also going to switch from using items.onloan to issues.due date at this point.  No special reason for that - just because.

{% highlight SQL linenos %}

SELECT
  items.homebranch,
  items.holdingbranch,
  permanent_locss.lib AS PERM_LOCATION,
  locss.lib AS LOCATION,
  itemtypess.description AS ITYPE,
  ccodes.lib AS CCODE,
  items.itemcallnumber,
  biblio.author,
  Concat_Ws(' ',
    biblio.title,
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="h"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="b"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="n"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="p"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="c"]')
  ) AS FULL_TITLE,
  items.barcode,
  items.itemnotes,
  items.itemnotes_nonpublic,
  items.issues,
  items.renewals,
  items.dateaccessioned,
  items.datelastborrowed,
  items.datelastseen,
  items.timestamp,
  issuesi.date_due,
  notforloans.lib,
  Concat(damageds.lib, ' ', items.damaged_on) AS DAMAGED,
  Concat(losts.lib, ' ', items.itemlost_on) AS LOST,
  Concat(withdrawns.lib, ' ', items.withdrawn_on) AS WITHDRAWN,

/* Creates a link to the bibliographic record  */

  If(
    issuesi.date_due IS NULL,
    "-",
    Concat(
      "<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=",
      issuesi.borrowernumber,
      "' target='_blank'>go to the borrower's account</a>"
    )
  ) AS LINK_TO_BORROWER


FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = items.itype LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn JOIN
  biblio_metadata ON biblio_metadata.biblionumber = biblio.biblionumber

/* Links items to the issues table so we can get current borrower information if the item is checked out */

LEFT JOIN
  (SELECT
      issues.itemnumber,
      issues.date_due,
      issues.borrowernumber
    FROM
      issues) issuesi ON issuesi.itemnumber = items.itemnumber


WHERE
  items.barcode LIKE Concat('%', <<Enter item barcode number>>, '%')
GROUP BY
  items.itemnumber,
  biblio.biblionumber

{% endhighlight %}

This piece:

{% highlight SQL linenos %}

Concat(
  "<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=",
  issuesi.borrowernumber,
  "' target='_blank'>go to the borrower's account</a>"
)

{% endhighlight %}

Creates the link to the borrower's record by combining some HTML with the borrower's ID number while the if/then statement sourrounding it tells the report to that, if there is no borrower number to report a hyphen instead of displaying an HTML link that won't work properly.

## Step 6

At this point I'm going to add in data from the branch transfers table so I can tell if the item is in transit between two libraries.

This involves another sub-query linking out to the branchtransfers table and listing items where branchtransfers.datearrived is null.  Once we have that we can get information about the from and to branches and concatenate them into the results.  Again, this is done through a left-join so that, if the item is not in transit, there will still be a result.  And I'm building it with an if/then statement so that, if the items isn't in transit, there will just be a hyphen instead of empty branchcodes saying "to" and "from."

{% highlight SQL linenos %}

SELECT
  items.homebranch,
  items.holdingbranch,
  permanent_locss.lib AS PERM_LOCATION,
  locss.lib AS LOCATION,
  itemtypess.description AS ITYPE,
  ccodes.lib AS CCODE,
  items.itemcallnumber,
  biblio.author,
  Concat_Ws(' ',
    biblio.title,
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="h"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="b"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="n"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="p"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="c"]')
  ) AS FULL_TITLE,
  items.barcode,
  items.itemnotes,
  items.itemnotes_nonpublic,
  items.issues,
  items.renewals,
  items.dateaccessioned,
  items.datelastborrowed,
  items.datelastseen,
  items.timestamp,
  issuesi.date_due,
  notforloans.lib,
  Concat(damageds.lib, ' ', items.damaged_on) AS DAMAGED,
  Concat(losts.lib, ' ', items.itemlost_on) AS LOST,
  Concat(withdrawns.lib, ' ', items.withdrawn_on) AS WITHDRAWN,

/* Shows if the item is in transit between libraries */

  If(
    transfersi.frombranch IS NULL,
    "-",
    Concat(transfersi.frombranch, " to ", transfersi.tobranch, " since ", transfersi.datesent)
  ) AS TRANSFER,


  If(
    issuesi.date_due IS NULL, "-",
    Concat(
      "<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=",
      issuesi.borrowernumber,
      "' target='_blank'>go to the borrower's account</a>"
    )
  ) AS LINK_TO_BORROWER
FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = items.itype LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn JOIN
  biblio_metadata ON biblio_metadata.biblionumber = biblio.biblionumber
  LEFT JOIN
  (SELECT
      issues.itemnumber,
      issues.date_due,
      issues.borrowernumber
    FROM
      issues) issuesi ON issuesi.itemnumber = items.itemnumber

/* Sub query to get data from the issues table */

LEFT JOIN
  (SELECT
      branchtransfers.itemnumber,
      branchtransfers.frombranch,
      branchtransfers.datesent,
      branchtransfers.tobranch,
      branchtransfers.datearrived
    FROM
      branchtransfers
    WHERE
      branchtransfers.datearrived IS NULL) transfersi ON transfersi.itemnumber =
      items.itemnumber


WHERE
  items.barcode LIKE Concat('%', <<Enter item barcode number>>, '%')
GROUP BY
  items.itemnumber,
  biblio.biblionumber

{% endhighlight %}

## Step 7

I also know that at some point I'm going to want to add labels to the data, so I'm going to do a single sample label with :

{% highlight SQL linenos %}

SELECT
  items.homebranch,
  items.holdingbranch,
  permanent_locss.lib AS PERM_LOCATION,
  locss.lib AS LOCATION,
  itemtypess.description AS ITYPE,
  ccodes.lib AS CCODE,
  items.itemcallnumber,
  biblio.author,
  Concat_Ws(' ',
    biblio.title,
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="h"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="b"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="n"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="p"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="c"]')
  ) AS FULL_TITLE,
  items.barcode,
  items.itemnotes,
  items.itemnotes_nonpublic,
  items.issues,
  items.renewals,
  items.dateaccessioned,
  items.datelastborrowed,
  items.datelastseen,
  items.timestamp,
  issuesi.date_due,
  notforloans.lib,
  Concat(damageds.lib, ' ', items.damaged_on) AS DAMAGED,
  Concat(losts.lib, ' ', items.itemlost_on) AS LOST,
  Concat(withdrawns.lib, ' ', items.withdrawn_on) AS WITHDRAWN,
  If(
    transfersi.frombranch IS NULL,
    "-",
    Concat(transfersi.frombranch, " to ", transfersi.tobranch, " since ", transfersi.datesent)
  ) AS TRANSFER,

/* Adds a label to the link to the borrower's account */

  Concat("Link to borrower: ",
    If(
      issuesi.date_due IS NULL,
      "-",
      Concat(
        "<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=",
        issuesi.borrowernumber,
        "' target='_blank'>go to the borrower's account</a>"
      )
    )
  ) AS LINK_TO_BORROWER


FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = items.itype LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn JOIN
  biblio_metadata ON biblio_metadata.biblionumber = biblio.biblionumber
  LEFT JOIN
  (SELECT
      issues.itemnumber,
      issues.date_due,
      issues.borrowernumber
    FROM
      issues) issuesi ON issuesi.itemnumber = items.itemnumber LEFT JOIN
  (SELECT
      branchtransfers.itemnumber,
      branchtransfers.frombranch,
      branchtransfers.datesent,
      branchtransfers.tobranch,
      branchtransfers.datearrived
    FROM
      branchtransfers
    WHERE
      branchtransfers.datearrived IS NULL) transfersi ON transfersi.itemnumber =
      items.itemnumber
WHERE
  items.barcode LIKE Concat('%', <<Enter item barcode number>>, '%')
GROUP BY
  items.itemnumber,
  biblio.biblionumber

{% endhighlight %}

By wrapping "Concat("Link to borrower: ", )" around the existing link to the borrower's information, I'm creating a lable that will be useful when we get to the future steps.

{% highlight SQL linenos %}

Concat("Link to borrower: ",
  If(
    issuesi.date_due IS NULL,
    "-",
    Concat(
      "<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=",
      issuesi.borrowernumber,
      "' target='_blank'>go to the borrower's account</a>"
    )
  )
) AS LINK_TO_BORROWER

{% endhighlight %}

## Step 8

Next I'm going to add in links with labels to other reports.  The important thing here is to have the reports on your system and to update the report numbers from what I have in my system to what you have in your system.  The reports I'm using are:

- Item circ history: 2785 - <a href="https://github.com/northeast-kansas-library-system/nextkansas.sql/blob/master/R.002785.sql" target="_blank">Click here for report 2785</a>
- Item action log history: 3342 - <a href="https://github.com/northeast-kansas-library-system/nextkansas.sql/blob/master/R.003342.sql" target="_blank">Click here for report 3342</a>
- Item in transit history: 2784 - <a href="https://github.com/northeast-kansas-library-system/nextkansas.sql/blob/master/R.002784.sql" target="_blank">Click here for report 2784</a>
- Request history on this title: 3039 - <a href="https://github.com/northeast-kansas-library-system/nextkansas.sql/blob/master/R.003039.sql" target="_blank">Click here for report 3039</a>
- Request history on this item: 3039 - <a href="https://github.com/northeast-kansas-library-system/nextkansas.sql/blob/master/R.003039.sql" target="_blank">Click here for report 3039</a>

I realize here that koha-US doesn't have a video on how to link results from one report to another report.  It's actually just like creating any other link as described in the video on links or at <a href="https://wiki.koha-community.org/wiki/SQL_Reports_Library#Links" target="_blank">the Koha Wiki's SQL library.</a>  The difference is that the URL will start with "/cgi-bin/koha/reports/guided_reports.pl?reports=", followed by the report number, followed by the runtime parameters you might need for the report.

For example, to run my "Item action log history" report you need to concatenate:

>"<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3342&phase=Run+this+report&param_name=Enter+item+number&sql_params="

to start the URL along with the itemnumber from the database followed by:

>"' target='_blank'>see action log history</a>"

to finish the HTML.

In this case the item number fills in the run time parameter the report is asking for.

Generally speaking, the best way to figure out how to concatenate report data into a report URL is to run that report with a sample set of runtime parameters and then copy and paste the URL into a text editor in order to figure out where the varialbes will go.  For example, if I run my "Request history on this item" with item barcode number 0003008200544, the resulting url is:

```

/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&param_name=Choose+pickup+library%7CLBRANCH&sql_params=%25&param_name=Choose+request+status%7CLHOLDACT&sql_params=%25&param_name=Choose+request+progress%7CLHOLDPROG&sql_params=%25&param_name=Choose+suspended+status%7CLHOLDSUS&sql_params=%25&param_name=Enter+library+card+number+or+a+%25+symbol&sql_params=%25&param_name=Enter+title+biblio+number+or+a+%25+symbol&sql_params=%25&param_name=Enter+item+barcode+number+or+a+%25+symbol&sql_params=0003008200544

```

and I can see the barcode number as the last 13 digits of that URL so I know that I would want to use everything up to the barcode number and then concatenate in items.barcode where the barcode number falls in this URL.

Anyway, there are plenty of examples to look at in this next version of the report:

{% highlight SQL linenos %}

SELECT
  items.homebranch,
  items.holdingbranch,
  permanent_locss.lib AS PERM_LOCATION,
  locss.lib AS LOCATION,
  itemtypess.description AS ITYPE,
  ccodes.lib AS CCODE,
  items.itemcallnumber,
  biblio.author,
  Concat_Ws(' ',
    biblio.title, ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="h"]'),
    ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="b"]'),
    ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="n"]'),
    ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="p"]'),
    ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="c"]')
  ) AS FULL_TITLE,
  items.barcode,
  items.itemnotes,
  items.itemnotes_nonpublic,
  items.issues,
  items.renewals,
  items.dateaccessioned,
  items.datelastborrowed,
  items.datelastseen,
  items.timestamp,
  issuesi.date_due,
  notforloans.lib,
  Concat(damageds.lib, ' ', items.damaged_on) AS DAMAGED,
  Concat(losts.lib, ' ', items.itemlost_on) AS LOST,
  Concat(withdrawns.lib, ' ', items.withdrawn_on) AS WITHDRAWN,

/* Adds a whole bunch of links with labels */

  Concat(
    "In transit from: ",
    If(
      transfersi.frombranch IS NULL,
      "-",
      Concat(transfersi.frombranch, " to ", transfersi.tobranch, " since ", transfersi.datesent)
    )
  ) AS TRANSFER,
  Concat(
    "Link to borrower: ",
    If(
      issuesi.date_due IS NULL,
      "-",
      Concat(
        "<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=",  
        issuesi.borrowernumber,
        "' target='_blank'>go to the borrower's account</a>"
      )
    )
  ) AS LINK_TO_BORROWER,
  Concat(
    "Link to title: ",
    Concat(
      "<a href='/cgi-bin/koha/catalogue/detail.pl?biblionumber=",
      biblio.biblionumber,
      "' target='_blank'>go to the bibliographic record</a>"
    )
  ) LINK_TO_TITLE,
  Concat(
    "Link to item: ",
    Concat(
      "<a href='/cgi-bin/koha/catalogue/moredetail.pl?itemnumber=",
      items.itemnumber,
      "&biblionumber=",
      biblio.biblionumber,
      "' target='_blank'>go to the item record</a>"
    )
  ) AS LINK_TO_ITEM,
  Concat(
    "Item circ history: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2785&phase=Run+this+report&param_name=Enter+item+barcode+number&sql_params=",
      items.barcode,
      "' target='_blank'>see item circ history</a>"
    )
  ) AS CIRC_HISTORY_REPORT,
  Concat(
    "Item action log history: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3342&phase=Run+this+report&param_name=Enter+item+number&sql_params=",
      items.itemnumber,
      "' target='_blank'>see action log history</a>"
    )
  ) AS ACTION_LOG,     
  Concat(
    "Item in transit history: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2784&phase=Run+this+report&sql_params=",
      items.barcode,
      "' target='_blank'>see item transit history</a>"
    )
  ) AS_IN_TRANSIT,
  Concat(
    "Request history on this title: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=",
      biblio.biblionumber,
      "&sql_params=%25' target='_blank'>see title's request history</a>"
    )
  ) AS REQUEST_HISTORY_TITLE,
  Concat(
    "Request history on this item: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=",
      items.barcode,
      "' target='_blank'>see item's request history</a>"
    )
  ) AS REQUEST_HISTORY_ITEM


FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = items.itype LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn JOIN
  biblio_metadata ON biblio_metadata.biblionumber = biblio.biblionumber
  LEFT JOIN
  (SELECT
      issues.itemnumber,
      issues.date_due,
      issues.borrowernumber
    FROM
      issues) issuesi ON issuesi.itemnumber = items.itemnumber LEFT JOIN
  (SELECT
      branchtransfers.itemnumber,
      branchtransfers.frombranch,
      branchtransfers.datesent,
      branchtransfers.tobranch,
      branchtransfers.datearrived
    FROM
      branchtransfers
    WHERE
      branchtransfers.datearrived IS NULL) transfersi ON transfersi.itemnumber =
      items.itemnumber
WHERE
  items.barcode LIKE Concat('%', <<Enter item barcode number>>, '%')
GROUP BY
  items.itemnumber,
  biblio.biblionumber

{% endhighlight %}

## Step 9

Now it's time to make this all look pretty by adding labels to everything.  I'm also adding "AS" statements to the end of each field so it's easy for anyone looking at this report what the fields I'm pulling from Koha are.  These "AS" statements aren't necessary, but I stuck them in here any way so it's perfectly clear what's going on.


{% highlight SQL linenos %}

SELECT


/* These changes just add labels to all of the fields that don't already have them */

  Concat("Home library: ", items.homebranch) AS HOMEBRANCH,
  Concat("Current library: ", items.holdingbranch) AS `CURRENT`,
  Concat("Permanent location: ", permanent_locss.lib) AS PERM_LOCATION,
  Concat("Current location: ", locss.lib) AS LOCATION,
  Concat("Item type: ", itemtypess.description) AS ITYPE,
  Concat("Collection code: ", ccodes.lib) AS CCODE,
  Concat("Call number: ", items.itemcallnumber) AS CALL_NUMBER,
  Concat("Author: ", biblio.author) AS AUTHOR,
  Concat("Title: ",
    Concat_Ws(' ',
      biblio.title, ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="h"]'),
      ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="b"]'),
      ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="n"]'),
      ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="p"]'),
      ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="c"]')
    )
  ) AS FULL_TITLE,
  Concat("Item barcode: ", items.barcode) AS BARCODE,
  Concat("Public notes: ", items.itemnotes) AS PUBLIC_NOTES,
  Concat("Non-public notes: ", items.itemnotes_nonpublic) AS NON_PUBLIC_NOTES,
  Concat("Checkouts: ", items.issues) AS CHECKOUT_COUNT,
  Concat("Renewals: ", items.renewals) AS RENEWAL_COUNT,
  Concat("Date added: ", items.dateaccessioned) AS DATE_ADDED,
  Concat("Last borrowed: ", items.datelastborrowed) AS LAST_BORROWED,
  Concat("Last seen: ", items.datelastseen) AS LAST_SEEN,
  Concat("Item record last modified: ", items.timestamp) AS LAST_MODIFIED,
  Concat("Due date: ", issuesi.date_due) AS DATE_DUE,
  Concat("Not for loan status: ", notforloans.lib) AS NFL,
  Concat("Damaged: ", Concat(damageds.lib, ' ', items.damaged_on)) AS DAMAGED,
  Concat("Lost: ", Concat(losts.lib, ' ', items.itemlost_on)) AS LOST,
  Concat("Withdrawn: ", Concat(withdrawns.lib, ' ', items.withdrawn_on)) AS WITHDRAWN,


  Concat(
    "In transit from: ",
    If(
      transfersi.frombranch IS NULL,
      "-",
      Concat(transfersi.frombranch, " to ", transfersi.tobranch, " since ", transfersi.datesent)
    )
  ) AS TRANSFER,
  Concat(
    "Link to borrower: ",
    If(
      issuesi.date_due IS NULL,
      "-",
      Concat(
        "<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=",  
        issuesi.borrowernumber,
        "' target='_blank'>go to the borrower's account</a>"
      )
    )
  ) AS LINK_TO_BORROWER,
  Concat(
    "Link to title: ",
    Concat(
      "<a href='/cgi-bin/koha/catalogue/detail.pl?biblionumber=",
      biblio.biblionumber,
      "' target='_blank'>go to the bibliographic record</a>"
    )
  ) LINK_TO_TITLE,
  Concat(
    "Link to item: ",
    Concat(
      "<a href='/cgi-bin/koha/catalogue/moredetail.pl?itemnumber=",
      items.itemnumber,
      "&biblionumber=",
      biblio.biblionumber,
      "' target='_blank'>go to the item record</a>"
    )
  ) AS LINK_TO_ITEM,
  Concat(
    "Item circ history: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2785&phase=Run+this+report&param_name=Enter+item+barcode+number&sql_params=",
      items.barcode,
      "' target='_blank'>see item circ history</a>"
    )
  ) AS CIRC_HISTORY_REPORT,
  Concat(
    "Item action log history: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3342&phase=Run+this+report&param_name=Enter+item+number&sql_params=",
      items.itemnumber,
      "' target='_blank'>see action log history</a>"
    )
  ) AS ACTION_LOG,     
  Concat(
    "Item in transit history: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2784&phase=Run+this+report&sql_params=",
      items.barcode,
      "' target='_blank'>see item transit history</a>"
    )
  ) AS_IN_TRANSIT,
  Concat(
    "Request history on this title: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=",
      biblio.biblionumber,
      "&sql_params=%25' target='_blank'>see title's request history</a>"
    )
  ) AS REQUEST_HISTORY_TITLE,
  Concat(
    "Request history on this item: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=",
      items.barcode,
      "' target='_blank'>see item's request history</a>"
    )
  ) AS REQUEST_HISTORY_ITEM
FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
     itemtypes.itemtype,
     itemtypes.description
   FROM
     itemtypes) itemtypess ON itemtypess.itemtype = items.itype LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn JOIN
  biblio_metadata ON biblio_metadata.biblionumber = biblio.biblionumber
  LEFT JOIN
  (SELECT
     issues.itemnumber,
     issues.date_due,
     issues.borrowernumber
   FROM
     issues) issuesi ON issuesi.itemnumber = items.itemnumber LEFT JOIN
  (SELECT
     branchtransfers.itemnumber,
     branchtransfers.frombranch,
     branchtransfers.datesent,
     branchtransfers.tobranch,
     branchtransfers.datearrived
   FROM
     branchtransfers
   WHERE
     branchtransfers.datearrived IS NULL) transfersi ON transfersi.itemnumber =
      items.itemnumber
WHERE
  items.barcode LIKE Concat('%', <<Enter item barcode number>>, '%')
GROUP BY
  items.itemnumber,
  biblio.biblionumber

{% endhighlight %}


## Step 10

The  report now has everything I want from the items table, but it's still not something I can cut-and-paste into an e-mail.  To get it there, I'm going to remove all of those "AS" statements and concatenate all of the fields into one field and I'm going to use CONCAT_WS and use an HTML line break as the seperator.


{% highlight SQL linenos %}

SELECT

/* This CONCAT_WS will turn every field into a separate line in one cell */

  CONCAT_WS("<br />",
    Concat("Home library: ", items.homebranch),
    Concat("Current library: ", items.holdingbranch),
    Concat("Permanent location: ", permanent_locss.lib),
    Concat("Current location: ", locss.lib),
    Concat("Item type: ", itemtypess.description),
    Concat("Collection code: ", ccodes.lib),
    Concat("Call number: ", items.itemcallnumber),
    Concat("Author: ", biblio.author),
    Concat("Title: ",
      Concat_Ws(' ',
        biblio.title, ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="h"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="b"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="n"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="p"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="c"]')
      )
    ),
    Concat("Item barcode: ", items.barcode),
    Concat("Public notes: ", items.itemnotes),
    Concat("Non-public notes: ", items.itemnotes_nonpublic),
    Concat("Checkouts: ", items.issues),
    Concat("Renewals: ", items.renewals),
    Concat("Date added: ", items.dateaccessioned),
    Concat("Last borrowed: ", items.datelastborrowed),
    Concat("Last seen: ", items.datelastseen),
    Concat("Item record last modified: ", items.timestamp),
    Concat("Due date: ", issuesi.date_due),
    Concat("Not for loan status: ", notforloans.lib),
    Concat("Damaged: ", Concat(damageds.lib, ' ', items.damaged_on)),
    Concat("Lost: ", Concat(losts.lib, ' ', items.itemlost_on)),
    Concat("Withdrawn: ", Concat(withdrawns.lib, ' ', items.withdrawn_on)),
    Concat(
      "In transit from: ",
      If(
        transfersi.frombranch IS NULL,
        "-",
        Concat(transfersi.frombranch, " to ", transfersi.tobranch, " since ", transfersi.datesent)
      )
    ),
    Concat(
      "Link to borrower: ",
      If(
        issuesi.date_due IS NULL,
        "-",
        Concat(
          "<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=",  
          issuesi.borrowernumber,
          "' target='_blank'>go to the borrower's account</a>"
        )
      )
    ),
    Concat(
      "Link to title: ",
      Concat(
        "<a href='/cgi-bin/koha/catalogue/detail.pl?biblionumber=",
        biblio.biblionumber,
        "' target='_blank'>go to the bibliographic record</a>"
      )
    ),
    Concat(
      "Link to item: ",
      Concat(
        "<a href='/cgi-bin/koha/catalogue/moredetail.pl?itemnumber=",
        items.itemnumber,
        "&biblionumber=",
        biblio.biblionumber,
        "' target='_blank'>go to the item record</a>"
      )
    ),
    Concat(
      "Item circ history: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2785&phase=Run+this+report&param_name=Enter+item+barcode+number&sql_params=",
        items.barcode,
        "' target='_blank'>see item circ history</a>"
      )
    ),
    Concat(
      "Item action log history: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3342&phase=Run+this+report&param_name=Enter+item+number&sql_params=",
        items.itemnumber,
        "' target='_blank'>see action log history</a>"
      )
    ),     
    Concat(
      "Item in transit history: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2784&phase=Run+this+report&sql_params=",
        items.barcode,
        "' target='_blank'>see item transit history</a>"
      )
    ),
    Concat(
      "Request history on this title: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=",
        biblio.biblionumber,
        "&sql_params=%25' target='_blank'>see title's request history</a>"
      )
    ),
    Concat(
      "Request history on this item: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=",
        items.barcode,
        "' target='_blank'>see item's request history</a>"
      )
    )
  ) AS INFO
FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
     itemtypes.itemtype,
     itemtypes.description
   FROM
     itemtypes) itemtypess ON itemtypess.itemtype = items.itype LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn JOIN
  biblio_metadata ON biblio_metadata.biblionumber = biblio.biblionumber
  LEFT JOIN
  (SELECT
     issues.itemnumber,
     issues.date_due,
     issues.borrowernumber
   FROM
     issues) issuesi ON issuesi.itemnumber = items.itemnumber LEFT JOIN
  (SELECT
     branchtransfers.itemnumber,
     branchtransfers.frombranch,
     branchtransfers.datesent,
     branchtransfers.tobranch,
     branchtransfers.datearrived
   FROM
     branchtransfers
   WHERE
     branchtransfers.datearrived IS NULL) transfersi ON transfersi.itemnumber =
      items.itemnumber
WHERE
  items.barcode LIKE Concat('%', <<Enter item barcode number>>, '%')
GROUP BY
  items.itemnumber,
  biblio.biblionumber

{% endhighlight %}

Now we have a report that looks like this:

![Screenshot - step 10 results](../images/1000.png)

## Step 11

Next I want to add a couple of additional line breaks to divide the parts of the output into sections.

{% highlight SQL linenos %}

SELECT
  CONCAT_WS("<br />",
    Concat("Home library: ", items.homebranch),
    Concat("Current library: ", items.holdingbranch),
    Concat("Permanent location: ", permanent_locss.lib),
    Concat("Current location: ", locss.lib),
    Concat("Item type: ", itemtypess.description),
    Concat("Collection code: ", ccodes.lib),
    Concat("Call number: ", items.itemcallnumber),
    Concat("Author: ", biblio.author),
    Concat("Title: ",
      Concat_Ws(' ',
        biblio.title, ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="h"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="b"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="n"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="p"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="c"]')
      )
    ),
    Concat("Item barcode: ", items.barcode),
    Concat("Public notes: ", items.itemnotes),
    Concat("Non-public notes: ", items.itemnotes_nonpublic),


/* Adding the "<br />" before the label adds a line break before these lines */

    Concat("<br />Checkouts: ", items.issues),
    Concat("Renewals: ", items.renewals),
    Concat("Date added: ", items.dateaccessioned),
    Concat("Last borrowed: ", items.datelastborrowed),
    Concat("Last seen: ", items.datelastseen),
    Concat("Item record last modified: ", items.timestamp),
    Concat("Due date: ", issuesi.date_due),
    Concat("Not for loan status: ", notforloans.lib),
    Concat("Damaged: ", Concat(damageds.lib, ' ', items.damaged_on)),
    Concat("Lost: ", Concat(losts.lib, ' ', items.itemlost_on)),
    Concat("Withdrawn: ", Concat(withdrawns.lib, ' ', items.withdrawn_on)),
    Concat(
      "In transit from: ",
      If(
        transfersi.frombranch IS NULL,
        "-",
        Concat(transfersi.frombranch, " to ", transfersi.tobranch, " since ", transfersi.datesent)
      )
    ),


/* Adding the "<br />" before the label adds a line break before these lines */

    Concat(
      "<br />Link to borrower: ",
      If(
        issuesi.date_due IS NULL,
        "-",
        Concat(
          "<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=",  
          issuesi.borrowernumber,
          "' target='_blank'>go to the borrower's account</a>"
        )
      )
    ),
    Concat(
      "Link to title: ",
      Concat(
        "<a href='/cgi-bin/koha/catalogue/detail.pl?biblionumber=",
        biblio.biblionumber,
        "' target='_blank'>go to the bibliographic record</a>"
      )
    ),
    Concat(
      "Link to item: ",
      Concat(
        "<a href='/cgi-bin/koha/catalogue/moredetail.pl?itemnumber=",
        items.itemnumber,
        "&biblionumber=",
        biblio.biblionumber,
        "' target='_blank'>go to the item record</a>"
      )
    ),


/* Adding the "<br />" before the label adds a line break before these lines */

    Concat(
      "<br />Item circ history: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2785&phase=Run+this+report&param_name=Enter+item+barcode+number&sql_params=",
        items.barcode,
        "' target='_blank'>see item circ history</a>"
      )
    ),
    Concat(
      "Item action log history: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3342&phase=Run+this+report&param_name=Enter+item+number&sql_params=",
        items.itemnumber,
        "' target='_blank'>see action log history</a>"
      )
    ),     
    Concat(
      "Item in transit history: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2784&phase=Run+this+report&sql_params=",
        items.barcode,
        "' target='_blank'>see item transit history</a>"
      )
    ),
    Concat(
      "Request history on this title: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=",
        biblio.biblionumber,
        "&sql_params=%25' target='_blank'>see title's request history</a>"
      )
    ),
    Concat(
      "Request history on this item: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=",
        items.barcode,
        "' target='_blank'>see item's request history</a>"
      )
    )
  ) AS INFO
FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
     itemtypes.itemtype,
     itemtypes.description
   FROM
     itemtypes) itemtypess ON itemtypess.itemtype = items.itype LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn JOIN
  biblio_metadata ON biblio_metadata.biblionumber = biblio.biblionumber
  LEFT JOIN
  (SELECT
     issues.itemnumber,
     issues.date_due,
     issues.borrowernumber
   FROM
     issues) issuesi ON issuesi.itemnumber = items.itemnumber LEFT JOIN
  (SELECT
     branchtransfers.itemnumber,
     branchtransfers.frombranch,
     branchtransfers.datesent,
     branchtransfers.tobranch,
     branchtransfers.datearrived
   FROM
     branchtransfers
   WHERE
     branchtransfers.datearrived IS NULL) transfersi ON transfersi.itemnumber =
      items.itemnumber
WHERE
  items.barcode LIKE Concat('%', <<Enter item barcode number>>, '%')
GROUP BY
  items.itemnumber,
  biblio.biblionumber

{% endhighlight %}

Now we have a report that looks like this:

![Screenshot - step 11 results](../images/1100.png)

## Step 12

And since I know that any items that this query will return will be items that are currently in the catalog, I'm going to concatenate in a header and a footer in the table that will make it easy for staff to recognize that this is a current item.

{% highlight SQL linenos %}

SELECT
  CONCAT_WS("<br />",

  /* This HTML will add a heading making it clear the item is active in the system */

    '<h3 style="color: white; background-color: #829356; text-align: center;">This item is currently in the catalog</h3>',


    Concat("Home library: ", items.homebranch),
    Concat("Current library: ", items.holdingbranch),
    Concat("Permanent location: ", permanent_locss.lib),
    Concat("Current location: ", locss.lib),
    Concat("Item type: ", itemtypess.description),
    Concat("Collection code: ", ccodes.lib),
    Concat("Call number: ", items.itemcallnumber),
    Concat("Author: ", biblio.author),
    Concat("Title: ",
      Concat_Ws(' ',
        biblio.title, ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="h"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="b"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="n"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="p"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="c"]')
      )
    ),
    Concat("Item barcode: ", items.barcode),
    Concat("Public notes: ", items.itemnotes),
    Concat("Non-public notes: ", items.itemnotes_nonpublic),
    Concat("<br />Checkouts: ", items.issues),
    Concat("Renewals: ", items.renewals),
    Concat("Date added: ", items.dateaccessioned),
    Concat("Last borrowed: ", items.datelastborrowed),
    Concat("Last seen: ", items.datelastseen),
    Concat("Item record last modified: ", items.timestamp),
    Concat("Due date: ", issuesi.date_due),
    Concat("Not for loan status: ", notforloans.lib),
    Concat("Damaged: ", Concat(damageds.lib, ' ', items.damaged_on)),
    Concat("Lost: ", Concat(losts.lib, ' ', items.itemlost_on)),
    Concat("Withdrawn: ", Concat(withdrawns.lib, ' ', items.withdrawn_on)),
    Concat(
      "In transit from: ",
      If(
        transfersi.frombranch IS NULL,
        "-",
        Concat(transfersi.frombranch, " to ", transfersi.tobranch, " since ", transfersi.datesent)
      )
    ),
    Concat(
      "<br />Link to borrower: ",
      If(
        issuesi.date_due IS NULL,
        "-",
        Concat(
          "<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=",  
          issuesi.borrowernumber,
          "' target='_blank'>go to the borrower's account</a>"
        )
      )
    ),
    Concat(
      "Link to title: ",
      Concat(
        "<a href='/cgi-bin/koha/catalogue/detail.pl?biblionumber=",
        biblio.biblionumber,
        "' target='_blank'>go to the bibliographic record</a>"
      )
    ),
    Concat(
      "Link to item: ",
      Concat(
        "<a href='/cgi-bin/koha/catalogue/moredetail.pl?itemnumber=",
        items.itemnumber,
        "&biblionumber=",
        biblio.biblionumber,
        "' target='_blank'>go to the item record</a>"
      )
    ),
    Concat(
      "<br />Item circ history: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2785&phase=Run+this+report&param_name=Enter+item+barcode+number&sql_params=",
        items.barcode,
        "' target='_blank'>see item circ history</a>"
      )
    ),
    Concat(
      "Item action log history: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3342&phase=Run+this+report&param_name=Enter+item+number&sql_params=",
        items.itemnumber,
        "' target='_blank'>see action log history</a>"
      )
    ),     
    Concat(
      "Item in transit history: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2784&phase=Run+this+report&sql_params=",
        items.barcode,
        "' target='_blank'>see item transit history</a>"
      )
    ),
    Concat(
      "Request history on this title: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=",
        biblio.biblionumber,
        "&sql_params=%25' target='_blank'>see title's request history</a>"
      )
    ),
    Concat(
      "Request history on this item: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=",
        items.barcode,
        "' target='_blank'>see item's request history</a>"
      )
    ),

    /* This HTML will add a footer making it clear the item is active in the system */

    '<br /><h3 style="color: white; background-color: #829356; text-align: center;">This item is currently in the catalog</h3>'

  ) AS INFO
FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
     itemtypes.itemtype,
     itemtypes.description
   FROM
     itemtypes) itemtypess ON itemtypess.itemtype = items.itype LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn JOIN
  biblio_metadata ON biblio_metadata.biblionumber = biblio.biblionumber
  LEFT JOIN
  (SELECT
     issues.itemnumber,
     issues.date_due,
     issues.borrowernumber
   FROM
     issues) issuesi ON issuesi.itemnumber = items.itemnumber LEFT JOIN
  (SELECT
     branchtransfers.itemnumber,
     branchtransfers.frombranch,
     branchtransfers.datesent,
     branchtransfers.tobranch,
     branchtransfers.datearrived
   FROM
     branchtransfers
   WHERE
     branchtransfers.datearrived IS NULL) transfersi ON transfersi.itemnumber =
      items.itemnumber
WHERE
  items.barcode LIKE Concat('%', <<Enter item barcode number>>, '%')
GROUP BY
  items.itemnumber,
  biblio.biblionumber

{% endhighlight %}

This half of the report now looks like this:

![Screenshot - step 12 results](../images/1200.png)

## Intermission

This half of the report is done.  I now have a report where I can enter an active item barcode number and I get a result that I can cut-and-paste into an e-mail and it quickly links to all of the reports that I may commonly need to run.

The next step is to write a report that runs, more-or-less, the same report against the deleteditems table instead of the items table.

I'm going to go through that in a lot fewer steps, though.

## Result I'm expecting

The result I'm eventually shooting for is this:

{% highlight SQL linenos %}

SELECT
  Concat_Ws('<br />',
  '<h2 style="color: white; background-color: #AD2A1A; text-align: center;">This item has been deleted</h2>',
  Concat(
    'At the time of its deletion on:  <ins><strong>',
    deleteditems.timestamp,
    "<br /></strong></ins> this item's information was as follows:<br />"
  ),
  Concat('Home library: ', deleteditems.homebranch),
  Concat('Current library: ', deleteditems.holdingbranch),
  Concat('Permanent location: ', deleteditems.permanent_location),
  Concat('Current location: ', deleteditems.location),
  Concat('Item type: ', deleteditems.itype),
  Concat('Collection code: ', ccodes.lib),
  Concat('Call#: ', deleteditems.itemcallnumber),
  Concat('Author: ', Coalesce(biblio.author, deletedbiblio.author)),
  Concat('Title: ', Coalesce(biblio.title, deletedbiblio.title)),
  Concat('Item barcode: ', deleteditems.barcode),
  Concat('Replacement price: ', deleteditems.replacementprice),
  Concat('Item id number: ', deleteditems.itemnumber),
  Concat(
    "<br />Damaged status: ",
    If(
      deleteditems.damaged = 0,
      "-",
      If(deleteditems.damaged IS NULL, "-", damagedi.lib)
    )
  ),
  Concat(
    "Lost status: ",
    If(
      deleteditems.itemlost = 0,
      "-",
      If(deleteditems.itemlost IS NULL, "-", Concat(losti.lib, " on ", deleteditems.itemlost_on))
    )
  ),
  Concat(
    "Withdrawn status: ",
    If(
      deleteditems.withdrawn = 0,
      "-",
      If(
        deleteditems.withdrawn IS NULL,
        "- ",
        Concat(deletedwithdrawni.lib, " on ", deleteditems.withdrawn_on)
      )
    )
  ),
  If(
    biblio.biblionumber IS NULL,
    "<br />-- Bibliographic record has been deleted --",
    Concat(
      "<br /><a href='/cgi-bin/koha/catalogue/detail.pl?biblionumber=",
      biblio.biblionumber,
      "' target='_blank'>Go to the bibliographic record</a>"
    )
  ),
  Concat(
    "<br /><a href='/cgi-bin/koha/reports/guided_reports.pl?phase=Run+this+report&reports=3009&sql_params=",
    Replace(
      Replace(
        Replace(
          Replace(
            Replace(
              Replace(
                Replace(
                  deleteditems.barcode,
                  Char(43),
                  "%2B"),
                Char(47),
                "%2F"),
              Char(32),
              "%20"),
            Char(45),
            "%2D"),
          Char(36),
          "%24"),
        Char(37),
        "%25"),
      Char(46),
      "%2E"
    ),
    "&limit=50' target='_blank'>Search payment and fee notes and descriptions for this item barcode number</a>"
  ),
  '<br /><h2 style="color: white; background-color: #AD2A1A; text-align: center;">This item was deleted from the catalog<br />within the past 13 months</h2>'
  ) AS INFO
FROM
  deleteditems
LEFT JOIN (
    SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE'
    ) ccodes
  ON deleteditems.ccode = ccodes.authorised_value
LEFT JOIN biblio
  ON deleteditems.biblionumber = biblio.biblionumber
LEFT JOIN deletedbiblio
  ON deleteditems.biblionumber = deletedbiblio.biblionumber
LEFT JOIN (
    SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED'
    ) damagedi
  ON damagedi.authorised_value = deleteditems.damaged
LEFT JOIN (
    SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST'
    ) losti
ON losti.authorised_value = deleteditems.itemlost
LEFT JOIN (
    SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN'
    ) deletedwithdrawni
ON deletedwithdrawni.authorised_value = deleteditems.withdrawn
WHERE
  deleteditems.barcode LIKE Concat("%", <<Enter item barcode number>>, "%")
GROUP BY
  deleteditems.itemnumber

{% endhighlight %}

And I'm going to take these steps to get there:


## Step 13

This SQL gets all of the data I might need from the deleted items table - but there is one major difference from the way I did this in the report for un-deleted items.  Instead of doing a normal join to the biblio table, I have to do a left join.  This is because, if an item has been deleted, it's possible that the corresponding bibliographic record has also been deleted.  If SQL is trying to show results where there must be a match between items.biblionumber and biblio.biblionumber and the bibliographic record no longer exists, you won't get any bibliogrpahic or item information.  If you use a left join, however, then you can get a results from the report even if the biblio has been moved to the deletedbiblio table.

To speed things up, I'm going to show this SQL at a point where I'm already getting the locations, item type descriptions, collection code information, and status information as descriptions instead of codes.  I'm also already combining the statuses and their dates where applicable.  I'm also dropping a few fields I don't need like "deleteditems.onloan."

{% highlight SQL linenos %}

SELECT
  deleteditems.homebranch,
  deleteditems.holdingbranch,
  deleteditems.permanent_location,
  deleteditems.location,
  deleteditems.itype,
  ccodes.lib,
  deleteditems.itemcallnumber,
  biblio.author,
  biblio.title,
  deleteditems.barcode,
  deleteditems.replacementprice,
  deleteditems.itemnumber,
  Concat(damageds.lib, ' ', deleteditems.damaged_on) AS DAMAGED,
  Concat(losts.lib, ' ', deleteditems.itemlost_on) AS LOST,
  Concat(withdrawns.lib, ' ', deleteditems.withdrawn_on) AS WITHDRAWN


/* Note here that the join between deleteditems and biblio is a LEFT JOIN */

FROM
  deleteditems LEFT JOIN
  biblio ON deleteditems.biblionumber = biblio.biblionumber

  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = deleteditems.permanent_location
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      deleteditems.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = deleteditems.itype
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      deleteditems.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = deleteditems.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = deleteditems.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      deleteditems.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = deleteditems.withdrawn
WHERE
  deleteditems.barcode LIKE Concat('%', <<Enter item barcode number>>, '%')
GROUP BY
  deleteditems.itemnumber,
  biblio.biblionumber

{% endhighlight %}


## Step 14

Since I can only get biblio information from the last step if the bibliographic record has not been deleted, the next logical step is to add in the deletedbiblio table so that I can get bibliographic information whether the biblio has been deleted or not.  This involves using the SQL "Coalesce" function.

If you don't know how to use coalesce, please check out the video: <a href="https://youtu.be/35UvrpcYFFA" target="_blank">SQL: Coalesce</a>

{% highlight SQL linenos %}

SELECT
deleteditems.homebranch,
deleteditems.holdingbranch,
deleteditems.permanent_location,
deleteditems.location,
deleteditems.itype,
ccodes.lib,
deleteditems.itemcallnumber,


/* Coalesce biblio and deletedbiblio data */

  Coalesce(biblio.author, deletedbiblio.author),
  Coalesce(biblio.title, deletedbiblio.title),


  deleteditems.barcode,
  deleteditems.replacementprice,
  deleteditems.itemnumber,
  Concat(damageds.lib, ' ', deleteditems.damaged_on) AS DAMAGED,
  Concat(losts.lib, ' ', deleteditems.itemlost_on) AS LOST,
  Concat(withdrawns.lib, ' ', deleteditems.withdrawn_on) AS WITHDRAWN
FROM
  deleteditems LEFT JOIN
  biblio ON deleteditems.biblionumber = biblio.biblionumber


/* New left join to deletedbiblio */

  LEFT JOIN
  deletedbiblio ON deletedbiblio.biblionumber = deleteditems.biblionumber LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = deleteditems.permanent_location
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      deleteditems.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = deleteditems.itype
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      deleteditems.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = deleteditems.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = deleteditems.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      deleteditems.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = deleteditems.withdrawn
WHERE
  deleteditems.barcode LIKE Concat('%', <<Enter item barcode number>>, '%')
GROUP BY
  deleteditems.itemnumber,
  biblio.biblionumber

{% endhighlight %}


## Step 15

There are some things that I don't have to do with the deleted items report that I did do with the still-available items report.  Specifically, I don't link out to item record and I don't link out to any of the same reports I was linking to before.  Those reports aren't built to show information about deleted items, so adding them to this report would really be a waste of time.

The page I do want to link out to link out to is the bibliographic record (if it hasn't been deleted).  I also have a report that looks for item barcode numbers in the accountlines descriptions and notes to help me identify fees and payments that might be associated with the deleted item.

That report is Item circ history: 3009 - <a href="https://github.com/northeast-kansas-library-system/nextkansas.sql/blob/master/R.003009.sql" target="_blank">Click here for report 2785</a>


{% highlight SQL linenos %}

SELECT
  deleteditems.homebranch,
  deleteditems.holdingbranch,
  deleteditems.permanent_location,
  deleteditems.location,
  deleteditems.itype,
  ccodes.lib,
  deleteditems.itemcallnumber,
  Coalesce(biblio.author, deletedbiblio.author),
  Coalesce(biblio.title, deletedbiblio.title),
  deleteditems.barcode,
  deleteditems.replacementprice,
  deleteditems.itemnumber,
  Concat(damageds.lib, ' ', deleteditems.damaged_on) AS DAMAGED,
  Concat(losts.lib, ' ', deleteditems.itemlost_on) AS LOST,
  Concat(withdrawns.lib, ' ', deleteditems.withdrawn_on) AS WITHDRAWN,


/* This section uses an if/then statement to display a message if the biblio has been deleted or a link to the biblio record if it has not */

  If(
    biblio.biblionumber IS NULL,
    "<br />-- Bibliographic record has been deleted --",
    Concat(
      "<br /><a href='/cgi-bin/koha/catalogue/detail.pl?biblionumber=",
      biblio.biblionumber,
      "' target='_blank'>Go to the bibliographic record</a>"
    )
  ),


/* This section links out to a report that searches for the item barcode number in accountlines */

  Concat(
    "<br /><a href='/cgi-bin/koha/reports/guided_reports.pl?phase=Run+this+report&reports=3009&sql_params=",
    Replace(
      Replace(
        Replace(
          Replace(
            Replace(
              Replace(
                Replace(
                  deleteditems.barcode,
                  Char(43),
                  "%2B"),
                Char(47),
                "%2F"),
              Char(32),
              "%20"),
            Char(45),
            "%2D"),
          Char(36),
          "%24"),
        Char(37),
        "%25"),
      Char(46),
      "%2E"
    ),
    "&limit=50' target='_blank'>Search payment and fee notes and descriptions for this item barcode number</a>"
  ) AS LINK_TO_REPORT



FROM
  deleteditems LEFT JOIN
  biblio ON deleteditems.biblionumber = biblio.biblionumber  LEFT JOIN
  deletedbiblio ON deletedbiblio.biblionumber = deleteditems.biblionumber LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = deleteditems.permanent_location
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      deleteditems.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = deleteditems.itype
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      deleteditems.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = deleteditems.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = deleteditems.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      deleteditems.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = deleteditems.withdrawn
WHERE
  deleteditems.barcode LIKE Concat('%', <<Enter item barcode number>>, '%')
GROUP BY
  deleteditems.itemnumber,
  biblio.biblionumber

{% endhighlight %}


## Step 16

And at this point I'm going to add all of the labels into the deleted items data.  Another thing you may notice, though, is that I've dropped the process of getting the 245$b, $p, $n, etc.  It is possible to get that data, but because you're looking in two different sets of tables, it's more complicated and not worth the effort for what I wanted to accomplish with deleted items.

{% highlight SQL linenos %}

SELECT
  Concat('Home library: ', deleteditems.homebranch),
  Concat('Current library: ', deleteditems.holdingbranch),
  Concat('Permanent location: ', deleteditems.permanent_location),
  Concat('Current location: ', deleteditems.location),
  Concat('Item type: ', deleteditems.itype),
  Concat('Collection code: ', ccodes.lib),
  Concat('Call#: ', deleteditems.itemcallnumber),
  Concat('Author: ', Coalesce(biblio.author, deletedbiblio.author)),
  Concat('Title: ', Coalesce(biblio.title, deletedbiblio.title)),
  Concat('Item barcode: ', deleteditems.barcode),
  Concat('Replacement price: ', deleteditems.replacementprice),
  Concat('Item id number: ', deleteditems.itemnumber),
  Concat(
    "<br />Damaged status: ",
    If(
      deleteditems.damaged = 0,
      "-",
      If(
        deleteditems.damaged IS NULL,
        "-",
        damageds.lib
      )
    )
  ),
  Concat(
    "Lost status: ",
    If(
      deleteditems.itemlost = 0,
      "-",
      If(
        deleteditems.itemlost IS NULL,
        "-",
        Concat(losts.lib, " on ", deleteditems.itemlost_on)
      )
    )
  ),
  Concat(
    "Withdrawn status: ",
    If(
      deleteditems.withdrawn = 0,
      "-",
      If(
        deleteditems.withdrawn IS NULL,
        "- ",
        Concat(withdrawns.lib, " on ", deleteditems.withdrawn_on)
      )
    )
  ),
  Concat(
    ": ",
    If(
      biblio.biblionumber IS NULL,
      "<br />-- Bibliographic record has been deleted --",
      Concat(
        "<br /><a href='/cgi-bin/koha/catalogue/detail.pl?biblionumber=",
        biblio.biblionumber,
        "' target='_blank'>Go to the bibliographic record</a>"
      )
    )
  ),
  Concat(
    "<br /><a href='/cgi-bin/koha/reports/guided_reports.pl?phase=Run+this+report&reports=3009&sql_params=",
    Replace(
      Replace(
        Replace(
          Replace(
            Replace(
              Replace(
                Replace(
                  deleteditems.barcode,
                  Char(43),
                  "%2B"),
                Char(47),
                "%2F"),
              Char(32),
              "%20"),
            Char(45),
            "%2D"),
          Char(36),
          "%24"),
        Char(37),
        "%25"),
      Char(46),
      "%2E"
    ),
    "&limit=50' target='_blank'>Search payment and fee notes and descriptions for this item barcode number</a>"
  )
FROM
  deleteditems LEFT JOIN
  biblio ON deleteditems.biblionumber = biblio.biblionumber LEFT JOIN
  deletedbiblio ON deletedbiblio.biblionumber = deleteditems.biblionumber
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = deleteditems.permanent_location
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      deleteditems.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = deleteditems.itype
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      deleteditems.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = deleteditems.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = deleteditems.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      deleteditems.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = deleteditems.withdrawn
WHERE
  deleteditems.barcode LIKE Concat('%', <<Enter item barcode number>>, '%')
GROUP BY
  deleteditems.itemnumber,
  biblio.biblionumber

{% endhighlight %}


## Step 17

The last step for the deleted item information, then, is to push it into one column with appropriate section breaks.  That looks like this:

{% highlight SQL linenos %}

SELECT


/* Here we add the concat_ws to put everything done so far into one cell in the results table */

  CONCAT_WS("<br />",

    Concat('Home library: ', deleteditems.homebranch),
    Concat('Current library: ', deleteditems.holdingbranch),
    Concat('Permanent location: ', deleteditems.permanent_location),
    Concat('Current location: ', deleteditems.location),
    Concat('Item type: ', deleteditems.itype),
    Concat('Collection code: ', ccodes.lib),
    Concat('Call#: ', deleteditems.itemcallnumber),
    Concat('Author: ', Coalesce(biblio.author, deletedbiblio.author)),
    Concat('Title: ', Coalesce(biblio.title, deletedbiblio.title)),
    Concat('Item barcode: ', deleteditems.barcode),
    Concat('Replacement price: ', deleteditems.replacementprice),
    Concat('Item id number: ', deleteditems.itemnumber),
    Concat(
      "<br />Damaged status: ",
      If(
        deleteditems.damaged = 0,
        "-",
        If(
          deleteditems.damaged IS NULL,
          "-",
          damageds.lib
        )
      )
    ),
    Concat(
      "Lost status: ",
      If(
        deleteditems.itemlost = 0,
        "-",
        If(
          deleteditems.itemlost IS NULL,
          "-",
          Concat(losts.lib, " on ", deleteditems.itemlost_on)
        )
      )
    ),
    Concat(
      "Withdrawn status: ",
      If(
        deleteditems.withdrawn = 0,
        "-",
        If(
          deleteditems.withdrawn IS NULL,
          "- ",
          Concat(withdrawns.lib, " on ", deleteditems.withdrawn_on)
        )
      )
    ),
    Concat(
      ": ",
      If(
        biblio.biblionumber IS NULL,
        "<br />-- Bibliographic record has been deleted --",
        Concat(
          "<br /><a href='/cgi-bin/koha/catalogue/detail.pl?biblionumber=",
          biblio.biblionumber,
          "' target='_blank'>Go to the bibliographic record</a>"
        )
      )
    ),
    Concat(
      "<br /><a href='/cgi-bin/koha/reports/guided_reports.pl?phase=Run+this+report&reports=3009&sql_params=",
      Replace(
        Replace(
          Replace(
            Replace(
              Replace(
                Replace(
                  Replace(
                    deleteditems.barcode,
                    Char(43),
                    "%2B"),
                  Char(47),
                  "%2F"),
                Char(32),
                "%20"),
              Char(45),
              "%2D"),
            Char(36),
            "%24"),
          Char(37),
          "%25"),
        Char(46),
        "%2E"
      ),
      "&limit=50' target='_blank'>Search payment and fee notes and descriptions for this item barcode number</a>"
    )

/* The ") AS INFO" closes out the CONCAT_WS at the beginning and gives the column a name */


  ) AS INFO
FROM
  deleteditems LEFT JOIN
  biblio ON deleteditems.biblionumber = biblio.biblionumber LEFT JOIN
  deletedbiblio ON deletedbiblio.biblionumber = deleteditems.biblionumber
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = deleteditems.permanent_location
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      deleteditems.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = deleteditems.itype
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      deleteditems.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = deleteditems.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = deleteditems.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      deleteditems.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = deleteditems.withdrawn
WHERE
  deleteditems.barcode LIKE Concat('%', <<Enter item barcode number>>, '%')
GROUP BY
  deleteditems.itemnumber,
  biblio.biblionumber

{% endhighlight %}


At this point we now have a report that will give us item information on an item that has been deleted but is still in the deleteditems table.

## Step 18

The last thing I'm going to do here is add a header and footer like I did for the active items.  In that SQL I used a green highlight.  In this one I'm going to use red.

{% highlight SQL linenos %}

SELECT
  CONCAT_WS("<br />",

  /* Here is the HTML that will make it obvious to staff that the item has been deleted */

    '<h2 style="color: white; background-color: #AD2A1A; text-align: center;">This item has been deleted</h2>',

    Concat('Home library: ', deleteditems.homebranch),
    Concat('Current library: ', deleteditems.holdingbranch),
    Concat('Permanent location: ', deleteditems.permanent_location),
    Concat('Current location: ', deleteditems.location),
    Concat('Item type: ', deleteditems.itype),
    Concat('Collection code: ', ccodes.lib),
    Concat('Call#: ', deleteditems.itemcallnumber),
    Concat('Author: ', Coalesce(biblio.author, deletedbiblio.author)),
    Concat('Title: ', Coalesce(biblio.title, deletedbiblio.title)),
    Concat('Item barcode: ', deleteditems.barcode),
    Concat('Replacement price: ', deleteditems.replacementprice),
    Concat('Item id number: ', deleteditems.itemnumber),
    Concat(
      "<br />Damaged status: ",
      If(
        deleteditems.damaged = 0,
        "-",
        If(
          deleteditems.damaged IS NULL,
          "-",
          damageds.lib
        )
      )
    ),
    Concat(
      "Lost status: ",
      If(
        deleteditems.itemlost = 0,
        "-",
        If(
          deleteditems.itemlost IS NULL,
          "-",
          Concat(losts.lib, " on ", deleteditems.itemlost_on)
        )
      )
    ),
    Concat(
      "Withdrawn status: ",
      If(
        deleteditems.withdrawn = 0,
        "-",
        If(
          deleteditems.withdrawn IS NULL,
          "- ",
          Concat(withdrawns.lib, " on ", deleteditems.withdrawn_on)
        )
      )
    ),
    Concat(
      ": ",
      If(
        biblio.biblionumber IS NULL,
        "<br />-- Bibliographic record has been deleted --",
        Concat(
          "<br /><a href='/cgi-bin/koha/catalogue/detail.pl?biblionumber=",
          biblio.biblionumber,
          "' target='_blank'>Go to the bibliographic record</a>"
        )
      )
    ),
    Concat(
      "<br /><a href='/cgi-bin/koha/reports/guided_reports.pl?phase=Run+this+report&reports=3009&sql_params=",
      Replace(
        Replace(
          Replace(
            Replace(
              Replace(
                Replace(
                  Replace(
                    deleteditems.barcode,
                    Char(43),
                    "%2B"),
                  Char(47),
                  "%2F"),
                Char(32),
                "%20"),
              Char(45),
              "%2D"),
            Char(36),
            "%24"),
          Char(37),
          "%25"),
        Char(46),
        "%2E"
      ),
      "&limit=50' target='_blank'>Search payment and fee notes and descriptions for this item barcode number</a>"
    ),

  /* Here is the HTML that will make it obvious to staff that the item has been deleted */

    '<br /><h2 style="color: white; background-color: #AD2A1A; text-align: center;">This item was deleted from the catalog<br />within the past 13 months</h2>'

  ) AS INFO
FROM
  deleteditems LEFT JOIN
  biblio ON deleteditems.biblionumber = biblio.biblionumber LEFT JOIN
  deletedbiblio ON deletedbiblio.biblionumber = deleteditems.biblionumber
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = deleteditems.permanent_location
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      deleteditems.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = deleteditems.itype
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      deleteditems.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = deleteditems.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = deleteditems.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      deleteditems.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = deleteditems.withdrawn
WHERE
  deleteditems.barcode LIKE Concat('%', <<Enter item barcode number>>, '%')
GROUP BY
  deleteditems.itemnumber,
  biblio.biblionumber

{% endhighlight %}


## Step 19

At this point we now have two working reports.  One that will take an active item barcode number and spit out a bunch of data and another that will take the barcode number from a deleted item and spit out a bunch of informaiton.  The final step is to combine the two reports.  This requires the SQL "Union" function.

If you don't know how to use "union", please check out the video: <a href="https://youtu.be/FJTFMaga-PU" target="_blank">SQL - Unions</a>

{% highlight SQL linenos %}

SELECT
  CONCAT_WS("<br />",
    '<h3 style="color: white; background-color: #829356; text-align: center;">This item is currently in the catalog</h3>',
    Concat("Home library: ", items.homebranch),
    Concat("Current library: ", items.holdingbranch),
    Concat("Permanent location: ", permanent_locss.lib),
    Concat("Current location: ", locss.lib),
    Concat("Item type: ", itemtypess.description),
    Concat("Collection code: ", ccodes.lib),
    Concat("Call number: ", items.itemcallnumber),
    Concat("Author: ", biblio.author),
    Concat("Title: ",
      Concat_Ws(' ',
        biblio.title, ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="h"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="b"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="n"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="p"]'),
        ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="c"]')
      )
    ),
    Concat("Item barcode: ", items.barcode),
    Concat("Public notes: ", items.itemnotes),
    Concat("Non-public notes: ", items.itemnotes_nonpublic),
    Concat("<br />Checkouts: ", items.issues),
    Concat("Renewals: ", items.renewals),
    Concat("Date added: ", items.dateaccessioned),
    Concat("Last borrowed: ", items.datelastborrowed),
    Concat("Last seen: ", items.datelastseen),
    Concat("Item record last modified: ", items.timestamp),
    Concat("Due date: ", issuesi.date_due),
    Concat("Not for loan status: ", notforloans.lib),
    Concat("Damaged: ", Concat(damageds.lib, ' ', items.damaged_on)),
    Concat("Lost: ", Concat(losts.lib, ' ', items.itemlost_on)),
    Concat("Withdrawn: ", Concat(withdrawns.lib, ' ', items.withdrawn_on)),
    Concat(
      "In transit from: ",
      If(
        transfersi.frombranch IS NULL,
        "-",
        Concat(transfersi.frombranch, " to ", transfersi.tobranch, " since ", transfersi.datesent)
      )
    ),
    Concat(
      "<br />Link to borrower: ",
      If(
        issuesi.date_due IS NULL,
        "-",
        Concat(
          "<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=",  
          issuesi.borrowernumber,
          "' target='_blank'>go to the borrower's account</a>"
        )
      )
    ),
    Concat(
      "Link to title: ",
      Concat(
        "<a href='/cgi-bin/koha/catalogue/detail.pl?biblionumber=",
        biblio.biblionumber,
        "' target='_blank'>go to the bibliographic record</a>"
      )
    ),
    Concat(
      "Link to item: ",
      Concat(
        "<a href='/cgi-bin/koha/catalogue/moredetail.pl?itemnumber=",
        items.itemnumber,
        "&biblionumber=",
        biblio.biblionumber,
        "' target='_blank'>go to the item record</a>"
      )
    ),
    Concat(
      "<br />Item circ history: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2785&phase=Run+this+report&param_name=Enter+item+barcode+number&sql_params=",
        items.barcode,
        "' target='_blank'>see item circ history</a>"
      )
    ),
    Concat(
      "Item action log history: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3342&phase=Run+this+report&param_name=Enter+item+number&sql_params=",
        items.itemnumber,
        "' target='_blank'>see action log history</a>"
      )
    ),     
    Concat(
      "Item in transit history: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2784&phase=Run+this+report&sql_params=",
        items.barcode,
        "' target='_blank'>see item transit history</a>"
      )
    ),
    Concat(
      "Request history on this title: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=",
        biblio.biblionumber,
        "&sql_params=%25' target='_blank'>see title's request history</a>"
      )
    ),
    Concat(
      "Request history on this item: ",
      Concat(
        "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=",
        items.barcode,
        "' target='_blank'>see item's request history</a>"
      )
    ),
    '<br /><h3 style="color: white; background-color: #829356; text-align: center;">This item is currently in the catalog</h3>'
  ) AS INFO
FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
     itemtypes.itemtype,
     itemtypes.description
   FROM
     itemtypes) itemtypess ON itemtypess.itemtype = items.itype LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
     authorised_values.category,
     authorised_values.authorised_value,
     authorised_values.lib
   FROM
     authorised_values
   WHERE
     authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn JOIN
  biblio_metadata ON biblio_metadata.biblionumber = biblio.biblionumber
  LEFT JOIN
  (SELECT
     issues.itemnumber,
     issues.date_due,
     issues.borrowernumber
   FROM
     issues) issuesi ON issuesi.itemnumber = items.itemnumber LEFT JOIN
  (SELECT
     branchtransfers.itemnumber,
     branchtransfers.frombranch,
     branchtransfers.datesent,
     branchtransfers.tobranch,
     branchtransfers.datearrived
   FROM
     branchtransfers
   WHERE
     branchtransfers.datearrived IS NULL) transfersi ON transfersi.itemnumber =
      items.itemnumber
WHERE
  items.barcode LIKE Concat('%', <<Enter item barcode number>>, '%')
GROUP BY
  items.itemnumber,
  biblio.biblionumber

/* UNION joins two reports together - they must have the same number of columns */

UNION



SELECT
  CONCAT_WS("<br />",
    '<h2 style="color: white; background-color: #AD2A1A; text-align: center;">This item has been deleted</h2>',
    Concat('Home library: ', deleteditems.homebranch),
    Concat('Current library: ', deleteditems.holdingbranch),
    Concat('Permanent location: ', deleteditems.permanent_location),
    Concat('Current location: ', deleteditems.location),
    Concat('Item type: ', deleteditems.itype),
    Concat('Collection code: ', ccodes.lib),
    Concat('Call#: ', deleteditems.itemcallnumber),
    Concat('Author: ', Coalesce(biblio.author, deletedbiblio.author)),
    Concat('Title: ', Coalesce(biblio.title, deletedbiblio.title)),
    Concat('Item barcode: ', deleteditems.barcode),
    Concat('Replacement price: ', deleteditems.replacementprice),
    Concat('Item id number: ', deleteditems.itemnumber),
    Concat(
      "<br />Damaged status: ",
      If(
        deleteditems.damaged = 0,
        "-",
        If(
          deleteditems.damaged IS NULL,
          "-",
          damageds.lib
        )
      )
    ),
    Concat(
      "Lost status: ",
      If(
        deleteditems.itemlost = 0,
        "-",
        If(
          deleteditems.itemlost IS NULL,
          "-",
          Concat(losts.lib, " on ", deleteditems.itemlost_on)
        )
      )
    ),
    Concat(
      "Withdrawn status: ",
      If(
        deleteditems.withdrawn = 0,
        "-",
        If(
          deleteditems.withdrawn IS NULL,
          "- ",
          Concat(withdrawns.lib, " on ", deleteditems.withdrawn_on)
        )
      )
    ),
    Concat(
      ": ",
      If(
        biblio.biblionumber IS NULL,
        "<br />-- Bibliographic record has been deleted --",
        Concat(
          "<br /><a href='/cgi-bin/koha/catalogue/detail.pl?biblionumber=",
          biblio.biblionumber,
          "' target='_blank'>Go to the bibliographic record</a>"
        )
      )
    ),
    Concat(
      "<br /><a href='/cgi-bin/koha/reports/guided_reports.pl?phase=Run+this+report&reports=3009&sql_params=",
      Replace(
        Replace(
          Replace(
            Replace(
              Replace(
                Replace(
                  Replace(
                    deleteditems.barcode,
                    Char(43),
                    "%2B"),
                  Char(47),
                  "%2F"),
                Char(32),
                "%20"),
              Char(45),
              "%2D"),
            Char(36),
            "%24"),
          Char(37),
          "%25"),
        Char(46),
        "%2E"
      ),
      "&limit=50' target='_blank'>Search payment and fee notes and descriptions for this item barcode number</a>"
    ),
    '<br /><h2 style="color: white; background-color: #AD2A1A; text-align: center;">This item was deleted from the catalog<br />within the past 13 months</h2>'
  ) AS INFO
FROM
  deleteditems LEFT JOIN
  biblio ON deleteditems.biblionumber = biblio.biblionumber LEFT JOIN
  deletedbiblio ON deletedbiblio.biblionumber = deleteditems.biblionumber
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = deleteditems.permanent_location
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      deleteditems.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = deleteditems.itype
  LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      deleteditems.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = deleteditems.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = deleteditems.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      deleteditems.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = deleteditems.withdrawn
WHERE
  deleteditems.barcode LIKE Concat('%', <<Enter item barcode number>>, '%')
GROUP BY
  deleteditems.itemnumber,
  biblio.biblionumber

{% endhighlight %}


## Conclusion

That's the whole thing.

It's a report that took a long time to create.  Like any truly lazy person, the goal of this report that took dozens of hours to create, was for the sake of saving 2 minutes when someone calls and says "Tell me about barcode number XX."

I have the report saved on my bookmarks toolbar so that when I do get a call saying "Tell me about this book?" I can just click on that bookmark and usually get them an answer in a moment or two.


-----
This work is licensed under a
[Creative Commons Attribution-ShareAlike 4.0 International License][cc-by-sa].

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png
[cc-by-sa-shield]: https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg
