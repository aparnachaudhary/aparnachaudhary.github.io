---
layout: post
title: Date DropDownChoice Component Apache Wicket
tags: [wicket, web-framework]
---

http://wicket.apache.org/\[Apache Wicket\] is yet another Java Web Development framework. But the beauty of Wicket is it provides clear separation of concerns. It doesn’t mix markup and java and saves you from adapting to additional expression language. Wicket templates are simple HTML files with only additional wicket attribute wicket:id. This makes it easy for the Java developers to work on the prototypes created by web designers. If you know core Java, its absolutely no problem adapting to Wicket programming model. Wicket comes with a nice quickstart guide and plenty of examples for using different components.

I just finished a project using Wicket. There are plenty of components available which we can just reuse. But in this project, we had a requirement to display date in Date, Month, Year drop down format instead of using simple text field with datepicker. I checked the wicket-datetime library, but sigh…the component was not available. I googled a bit and found the nice implementation by Mystic Coders. A small TODO thing in this piece of code is that the date drop down always show 31 days irrespective of the month and year. So I did some minor enhancement to allow population of date drop down based on Month and Year(Leap year consideration).

In the DateChooser.java, first of all change the getDays() function to return days based on month and year. Secondly, we need to add the AjaxFormComponentUpdatingBehavior to month and year DDC. Make sure you set the output markup id for dayDDC to true.

```java

package nl.jteam.common.wicket.component;

import java.text.SimpleDateFormat; import java.util.ArrayList; import java.util.Calendar; import java.util.Date; import java.util.GregorianCalendar; import java.util.List;

import org.apache.wicket.WicketRuntimeException; import org.apache.wicket.ajax.AjaxRequestTarget; import org.apache.wicket.ajax.form.AjaxFormComponentUpdatingBehavior; import org.apache.wicket.markup.html.form.DropDownChoice; import org.apache.wicket.markup.html.form.FormComponent; import org.apache.wicket.markup.html.form.IChoiceRenderer; import org.apache.wicket.model.IModel; import org.apache.wicket.model.Model;

/\*\* \* DateChooser \* &lt;p/&gt; \* Created by: Andrew Lombardi Copyright 2006 Mystic Coders, LLC \* &lt;p/&gt; \* With helpful contributions from: &lt;a \* href="http://www.systemmobile.com"&gt;mr\_smith&lt;/a&gt; &lt;a \* href="http://www.almaw.com"&gt;AlMaw&lt;/a&gt; and ivaynberg \* &lt;p&gt; \* Aparna: Overridden convertInput() method and added logic to get days based on \* month and year. \* &lt;/p&gt; * * @author Aparna Chaudhary \*/ public class DateChooser extends FormComponent&lt;Date&gt; {

/\*\* The day ddc. \*/ private DropDownChoice&lt;Integer&gt; dayDDC;

/\*\* The month ddc. \*/ private DropDownChoice&lt;Integer&gt; monthDDC;

/\*\* The year ddc. \*/ private DropDownChoice&lt;Integer&gt; yearDDC;

/\*\* \* Instantiates a new date chooser. * * @param id the id \* @param model the model \*/ public DateChooser(final String id, IModel&lt;Date&gt; model) { super(id);

if (model == null) { model = new Model&lt;Date&gt;(new Date()); } else if (model.getObject() == null) { model.setObject(new Date()); } else if (!(model.getObject() instanceof Date)) { throw new WicketRuntimeException("DateChooser \[" + getPath() + "\] contains an invalid model object, must be an object of type java.util.Date"); }

setModel(model);

monthDDC = new DropDownChoice&lt;Integer&gt;("month", new DateModel(model, Calendar.MONTH), getMonths(), new IChoiceRenderer&lt;Integer&gt;() {

public Object getDisplayValue(Integer object) { SimpleDateFormat format = new SimpleDateFormat("MMM");

Calendar cal = Calendar.getInstance(); cal.set(Calendar.MONTH, object.intValue());

return format.format(cal.getTime()); }

public String getIdValue(Integer integer, int index) { return String.valueOf(index); } }); monthDDC.add(new AjaxFormComponentUpdatingBehavior("onchange") { protected void onUpdate(AjaxRequestTarget target) { // change the days dropdown when month changes dayDDC.setChoices(getDays()); target.addComponent(dayDDC); } }); add(monthDDC);

yearDDC = new DropDownChoice&lt;Integer&gt;("year", new DateModel(model, Calendar.YEAR), getYears()); add(yearDDC); yearDDC.add(new AjaxFormComponentUpdatingBehavior("onchange") { protected void onUpdate(AjaxRequestTarget target) { // change the days dropdown when year changes dayDDC.setChoices(getDays()); target.addComponent(dayDDC); } });

dayDDC = new DropDownChoice&lt;Integer&gt;("day", new DateModel(model, Calendar.DAY\_OF\_MONTH), getDays()); dayDDC.setOutputMarkupId(true); add(dayDDC); }

/\*\* \* The Class DateModel. \*/ private class DateModel extends Model&lt;Integer&gt; {

/\*\* The date model. \*/ private final IModel&lt;Date&gt; dateModel;

/\*\* The calendar field. \*/ private final int calendarField;

/\*\* \* Instantiates a new date model. * * @param dateModel the date model \* @param calendarField the calendar field \*/ public DateModel(IModel&lt;Date&gt; dateModel, int calendarField) { this.dateModel = dateModel; this.calendarField = calendarField; }

/* * (non-Javadoc) \* @see org.apache.wicket.model.Model\#detach() \*/ @Override public void detach() { dateModel.detach(); }

/* * (non-Javadoc) \* @see org.apache.wicket.model.Model\#getObject() \*/ @Override public Integer getObject() { if (dateModel.getObject() == null) { return null; }

Calendar cal = Calendar.getInstance(); cal.setTime(dateModel.getObject());

return cal.get(calendarField); }

/* * (non-Javadoc) \* @see org.apache.wicket.model.Model\#setObject(java.io.Serializable) \*/ @Override public void setObject(Integer object) { Date date = dateModel.getObject(); if (date == null) { date = new Date(); } Calendar cal = Calendar.getInstance(); cal.setTime(date); cal.set(calendarField, object);

dateModel.setObject(cal.getTime()); } }

/\*\* \* Gets the months. * * @return the months \*/ private List&lt;Integer&gt; getMonths() { List&lt;Integer&gt; months = new ArrayList&lt;Integer&gt;(12);

for (int i = 0; i &lt; 12; i++) { months.add(i); }

return months; }

/\*\* \* Gets the days. * * @return the days \*/ protected List&lt;Integer&gt; getDays() { List&lt;Integer&gt; days = new ArrayList&lt;Integer&gt;(31); int totalDays = 31; if (yearDDC.getModelObject() != null && monthDDC.getModelObject() != null) { Calendar cal = new GregorianCalendar(yearDDC.getModelObject(), monthDDC.getModelObject(), 1); totalDays = cal.getActualMaximum(Calendar.DAY\_OF\_MONTH); } for (int i = 1; i &lt;= totalDays; i++) { days.add(i); } return days; }

/\*\* \* Gets the years. * * @return the years \*/ protected List&lt;Integer&gt; getYears() { List&lt;Integer&gt; years = new ArrayList&lt;Integer&gt;(10);

Calendar cal = Calendar.getInstance();

for (int i = cal.get(Calendar.YEAR) - 2; i &lt; cal.get(Calendar.YEAR) + 8; i++) { years.add(i); }

return years; }

/* * (non-Javadoc) \* @see org.apache.wicket.markup.html.form.FormComponent\#convertInput() \*/ @Override protected void convertInput() { Calendar cal = Calendar.getInstance(); cal.set(yearDDC.getConvertedInput(), monthDDC.getConvertedInput(), dayDDC.getConvertedInput(), 0, 0, 0); Date selectedDate = cal.getTime(); setModel(new Model&lt;Date&gt;(selectedDate)); }

}
```

The above component is tested with *Wicket-1.4-rc5*.
