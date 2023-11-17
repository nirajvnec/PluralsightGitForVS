// Assuming you have already defined and initialized your TableLayoutPanel
// and it's named tableLayoutPanel1 in your form designer code

// Adding a new ProgressBar control
ProgressBar progressBar = new ProgressBar();
progressBar.Name = "progressBar";
progressBar.Dock = DockStyle.Fill;

// Add a new RowStyle for the ProgressBar at the second row (index 1)
// If your TableLayoutPanel does not have a second row, you need to add one
tableLayoutPanel1.RowStyles.Add(new RowStyle(SizeType.AutoSize));
tableLayoutPanel1.RowCount = 2; // Set the row count to 2
tableLayoutPanel1.Controls.Add(progressBar, 0, 1); // Add progressBar to the new row

// Set the column span if you want the ProgressBar to span across multiple columns
tableLayoutPanel1.SetColumnSpan(progressBar, tableLayoutPanel1.ColumnCount);

// Now you have your ProgressBar added to the second row of your TableLayoutPanel







private async void m_searchControlSearchEvent(object sender, SearchRiskAttributeEventArgs e)
{
    pnlSearch.Visible = true;
    breakdown_attributes.Visible = false;
    breakdown_headings_combo_box.Enabled = false;
    breakdown_headings_combo_box.SelectedIndex = 1;
    DisableBreakdownsByText(m_global_cache.HideBreakdownAndDrilldowns);

    var selectedCalculationMethod = this.GetSelectedMethod();

    if (ctlCalculationMethod.SelectedMethod is CsCalculationContribution && 
        ((CsCalculationContribution)ctlCalculationMethod.SelectedMethod).IsErcType)
    {
        if (rbtnConstant.Checked)
        {
            RemoveColumnRowBreakdown("cobdate", CobDateBreakdown);
            DisableBreakdownByText(CobDateBreakdown, true);
        }
        else
        {
            DisableBreakdownChooseDetailsOption(CobDateBreakdown, false);
            DisableBreakdownByText(RISK_TYPE, false);
        }
    }
    else
    {
        DisableBreakdownByText(CobDateBreakdown, false);
        DisableBreakdownChooseDetailsOption(CobDateBreakdown, true);

        if (!(selectedCalculationMethod is CsCalculationMethodBMCPlStrip) && 
            !(selectedCalculationMethod is CsCalculationMethodRNIVMarginalToVar))
        {
            DisableBreakdownByText(RISK_TYPE, false);
        }
    }

    var progress = new Progress<int>(percent =>
    {
        // Update the UI based on progress
        // For example, updating a progress bar
        // progressBar.Value = percent;
    });

    // Modify the SearchAttributes method to support Progress<T> and call it here
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

    this.Invoke(new MethodInvoker(() => {
        this.SuspendLayout();
        label_area_panel.Top = TopOfLabelArea;
        lbl_separator.Visible = false;
        m_drag_drop_items.SuspendLayout();
    }));

    int totalItems = m_drag_drop_items.Count;
    int processedItems = 0;

    foreach (CtlDragDropItem drag_drop_item in m_drag_drop_items)
    {
        bool isSubString = false;
        this.Invoke(new MethodInvoker(() => {
            if (IsHidden(((MarsEnquiryTool.CtlDragDropBreakdown)drag_drop_item))) return;
            if (HideErcNtcsMnpiAttributes(((MarsEnquiryTool.CtlDragDropBreakdown)drag_drop_item))) return;

            if (!drag_drop_item.IsDraggedAway)
            {
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
        }));

        processedItems++;
        progress?.Report((processedItems * 100) / totalItems);
    }

    this.Invoke(new MethodInvoker(() => {
        m_drag_drop_items.ResumeLayout(false);
        label_area_panel.Height = label_top;
        RedrawScrollArea();
        this.ResumeLayout(false);
    }));

    return resultfound;
}
