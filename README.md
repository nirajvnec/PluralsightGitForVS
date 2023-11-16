private async void m_searchControl_SearchEvent(object sender, SearchRiskAttributeEventArgs e)
{
    pnlSearch.Visible = true;
    breakdown_attributes.Visible = false;
    breakdown_headings_combo_box.Enabled = false;
    breakdown_headings_combo_box.SelectedIndex = 1;
    DisableBreakdownsByText(m_global_cache.HideBreakdownAndDrilldowns);

    var progress = new Progress<int>(percent =>
    {
        // Update UI based on progress, e.g., updating a progress bar or a label
        // Example: progressBar.Value = percent;
    });

    await SearchAttributes(e, progress);

    breakdown_attributes.Visible = true;
    pnlSearch.Visible = false;
}



private async Task SearchAttributes(SearchRiskAttributeEventArgs e, IProgress<int> progress)
{
    string searchString = e.RiskAttributeName.ToUpper();
    bool resultFound = await Task.Run(() => RedrawSearchItems(searchString, progress));

    // Update the UI based on the result of the search
    if (!resultFound)
    {
        this.Invoke(new Action(() =>
        {
            empty_text_label.Visible = true;
            empty_text_label.Text = "No matching attributes found...";
        }));
    }
    else
    {
        this.Invoke(new Action(() =>
        {
            empty_text_label.Visible = false;
        }));
    }
}


public bool RedrawSearchItems(string searchstring, IProgress<int> progress)
{
    int label_top = SIZE_ITEM_SPACING;
    bool resultfound = false;
    string[] searchStringArray = searchstring.Split(new Char[] { ' ', '+', '-', '_', '(', ')', '#', '@', '{', '}', '[', ']', '*', '|'}, StringSplitOptions.RemoveEmptyEntries);

    this.SuspendLayout();
    label_area_panel.Top = TopOfLabelArea;
    lbl_separator.Visible = false;
    m_drag_drop_items.SuspendLayout();

    int totalItems = m_drag_drop_items.Count;
    int processedItems = 0;

    foreach (CtlDragDropItem drag_drop_item in m_drag_drop_items)
    {
        if (IsHidden(((MarsEnquiryTool.CtlDragDropBreakdown)drag_drop_item))) continue;
        if (HideErcNtcsMnpiAttributes(((MarsEnquiryTool.CtlDragDropBreakdown)drag_drop_item))) continue;

        if (!drag_drop_item.IsDraggedAway)
        {
            isSubString = false;
            foreach (string searchSubstring in searchStringArray)
            {
                isSubString = drag_drop_item.Text.ToUpper().StartsWith(searchSubstring.ToUpper());
                if (isSubString) break;
            }

            if (!drag_drop_item.Disabled && isSubString)
            {
                drag_drop_item.Top = label_top;
                label_top += SIZE_ITEM_SPACING + drag_drop_item.Height;
                drag_drop_item.Visible = true;
                resultfound = true;
            }
            else
            {
                drag_drop_item.Visible = false;
            }
        }

        processedItems++;
        progress?.Report((processedItems * 100) / totalItems);
    }

    m_drag_drop_items.ResumeLayout();
    label_area_panel.Height = label_top;
    RedrawScrollArea();
    this.ResumeLayout();

    return resultfound;
}
