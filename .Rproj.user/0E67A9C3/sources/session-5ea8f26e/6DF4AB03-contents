#################################################
# LIBRARIES
#################################################
library(shiny)
library(shinydashboard)
library(ggplot2)
library(dplyr)
library(readr)

install.packages(c("shiny", "shinydashboard", "ggplot2", "dplyr", "readr", "remotes"))
remotes::install_github("R-CoderDotCom/ggdogs")

# Safety check BEFORE loading ggdogs
if (!requireNamespace("ggdogs", quietly = TRUE)) {
  stop("Package 'ggdogs' is required. Install with remotes::install_github('R-CoderDotCom/ggdogs')")
}

library(ggdogs)

#################################################
# UI
#################################################

ui <- dashboardPage(
  dashboardHeader(title = "Dog Boxplot App"),
  
  dashboardSidebar(
    
    fileInput(
      "file1",
      "Upload CSV File max 5 MB",
      accept = ".csv"
    ),
    
    selectInput(
      "xvar",
      "Grouping variable:",
      choices = NULL
    ),
    
    selectInput(
      "yvar",
      "Numeric variable:",
      choices = NULL
    ),
    
    # NUEVO: Campo para el título del gráfico
    textInput(
      "plot_title",
      "Plot title:",
      value = "Boxplot with Dog Outliers",
      placeholder = "Type your custom title here..."
    ),
    
    radioButtons(
      "orientation",
      "Plot orientation:",
      choices = c("Vertical" = "vertical",
                  "Horizontal" = "horizontal"),
      selected = "vertical",
      inline = TRUE
    ),
    
    selectInput(
      "dog",
      "Dog style:",
      choices = c("doge","doge_strong","chihuahua",
                  "eyes","gabe","glasses",
                  "tail","surprised","thisisfine",
                  "hearing","pug","ears",
                  "husky","husky_2","chilaquil")
    ),
    
    sliderInput(
      "size",
      "Dog size:",
      min = 1,
      max = 10,
      value = 3
    ),
    
    selectInput(
      "download_format",
      "Download format:",
      choices = c("PNG" = "png",
                  "JPG" = "jpg",
                  "PDF" = "pdf"),
      selected = "png"
    ),
    
    downloadButton("download_plot", "Download Plot")
    
  ),
  
  dashboardBody(
    
    fluidRow(
      
      # ==========================================
      # 👇 INSTRUCTIONS HTML STARTS HERE 👇
      # ==========================================
      box(
        width = 12,
        title = "🐶 CheemsPlot: User Instructions",
        status = "primary",
        solidHeader = FALSE,
        collapsible = TRUE,
        collapsed = FALSE,
        
        HTML(
          "<h4><b>Welcome to the Dog Boxplot Explorer</b></h4>
          <p>This app turns traditional boxplots into a fun visual experience by replacing outliers with dog icons using the <code>ggdogs</code> package.</p>
          
          <h4><b>📂 How to Use the App</b></h4>
          <ul>
            <li><b>Step 1:</b> Upload a <b>CSV</b> file using the 'Browse...' button on the left panel.</li>
            <li><b>Step 2:</b> Select the <b>Grouping variable</b> (must be categorical, e.g. breeds, groups).</li>
            <li><b>Step 3:</b> Select the <b>Numeric variable</b> (the values you want to plot).</li>
            <li><b>Step 4:</b> Type a custom <b>Plot title</b> (or keep the default).</li>
            <li><b>Step 5:</b> Choose the <b>Plot orientation</b> (Vertical or Horizontal).</li>
            <li><b>Step 6:</b> Choose the <b>Dog style</b> that will appear on the outliers.</li>
            <li><b>Step 7:</b> Adjust the <b>Dog size</b> with the slider.</li>
            <li><b>Step 8:</b> Choose the <b>Download format</b> and click <b>Download Plot</b>.</li>
          </ul>

          <h4><b>📊 What Are You Looking At?</b></h4>
          <p>The plot shows the distribution of your data. Points falling outside the interquartile range (IQR) are considered <b>outliers</b> and are represented with the selected dog. Style up your analysis!</p>
          
          <h4><b>📋 CSV File Requirements</b></h4>
          <ul>
            <li>Must contain at least one <b>categorical</b> column (text) for grouping.</li>
            <li>Must contain at least one <b>numeric</b> column for the Y axis.</li>
          </ul>
          
          <hr>
          <p><i>Inspired by the creativity of the R community. Author: Ar Sullivan Santana Najera.</i></p>"
        )
      ),
      # ==========================================
      # 👆 INSTRUCTIONS HTML ENDS HERE 👆
      # ==========================================
      
      box(
        width = 12,
        plotOutput("boxplot_dogs", height = "600px")
      )
    )
    
  )
)

#################################################
# SERVER
#################################################

server <- function(input, output, session) {
  
  data_uploaded <- reactive({
    req(input$file1)
    read_csv(input$file1$datapath)
  })
  
  observe({
    df <- data_uploaded()
    numeric_cols <- names(df)[sapply(df, is.numeric)]
    factor_cols <- names(df)[!sapply(df, is.numeric)]
    updateSelectInput(session, "xvar", choices = factor_cols)
    updateSelectInput(session, "yvar", choices = numeric_cols)
  })
  
  # Función reactiva que construye el gráfico (reutilizable)
  build_plot <- reactive({
    req(input$xvar, input$yvar)
    
    df <- data_uploaded()
    x <- input$xvar
    y <- input$yvar
    
    outliers <- df %>%
      group_by(.data[[x]]) %>%
      mutate(
        Q1 = quantile(.data[[y]], 0.25, na.rm = TRUE),
        Q3 = quantile(.data[[y]], 0.75, na.rm = TRUE),
        IQR = Q3 - Q1,
        lower = Q1 - 1.5 * IQR,
        upper = Q3 + 1.5 * IQR
      ) %>%
      filter(.data[[y]] < lower | .data[[y]] > upper)
    
    # Si el usuario deja el título vacío, usamos uno por defecto
    titulo <- ifelse(
      is.null(input$plot_title) || trimws(input$plot_title) == "",
      "Boxplot with Dog Outliers",
      input$plot_title
    )
    
    p <- ggplot(df, aes(x = .data[[x]], y = .data[[y]])) +
      geom_boxplot(outlier.shape = NA, fill = "lightblue") +
      geom_dog(
        data = outliers,
        aes(x = .data[[x]], y = .data[[y]]),
        dog = input$dog,
        size = input$size
      ) +
      theme_minimal(base_size = 16) +
      labs(
        title = titulo,   # <-- Ahora usa el título del usuario
        x = x,
        y = y
      )
    
    if (input$orientation == "horizontal") {
      p <- p + coord_flip()
    }
    
    p
  })
  
  # Render del gráfico en pantalla
  output$boxplot_dogs <- renderPlot({
    build_plot()
  })
  
  # Descarga del gráfico
  output$download_plot <- downloadHandler(
    filename = function() {
      paste0("dog_boxplot_", Sys.Date(), ".", input$download_format)
    },
    content = function(file) {
      ggsave(
        filename = file,
        plot = build_plot(),
        device = input$download_format,
        width = 10,
        height = 7,
        dpi = 300
      )
    }
  )
  
}

#################################################
# RUN APP
#################################################
shinyApp(ui, server)