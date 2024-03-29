---
title: SwiftUI4：MapView
author: 独孤流
date: 2024-03-20 01:01:00 +0800
categories: [SwiftUI]
tags: [SwiftUI]     # TAG names should always be lowercase
---

参考：
- [[SwiftUI 知识碎片] 在 SwiftUI 中使用 MapKit](https://juejin.cn/post/6844904078279966727)
- [[SwiftUI 知识碎片] 自定义 MKMapView 标记](https://juejin.cn/post/6844904083434782734)
- [StudyMapView.swift](https://github.com/h42330789/StudySwiftUI/blob/main/StudySwiftUI/StudySwiftUI/StudyMapView.swift)

本文主要参考学习使用MKMapView的使用，以及`SwiftUI`里的`View`与`UIKit`里的`UIView`进行交互

默认的标注
自定义的标注
```
import SwiftUI
import MapKit

struct MyMapView: UIViewRepresentable {
    typealias UIViewType = MKMapView
    @Binding var centerCoordinate: CLLocationCoordinate2D
    var annotations: [MKPointAnnotation]
    @Binding var selectedPlace: MKPointAnnotation?
    @Binding var showingPlaceDetails: Bool
    
    func makeUIView(context: Context) -> MKMapView {
        let mapView = MKMapView()
        mapView.delegate = context.coordinator
        return mapView
    }
    
    func updateUIView(_ view: MKMapView, context: Context) {
        if annotations.count != view.annotations.count {
                view.removeAnnotations(view.annotations)
                view.addAnnotations(annotations)
            }
    }
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    

}

class Coordinator: NSObject, MKMapViewDelegate {
    var parent: MyMapView
    
    init(_ parent: MyMapView) {
        self.parent = parent
    }
    
    func mapViewDidChangeVisibleRegion(_ mapView: MKMapView) {
        parent.centerCoordinate = mapView.centerCoordinate
    }
    
    func mapView(_ mapView: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView? {
        let identifier = "Placemark"
        
        var annotationView = mapView.dequeueReusableAnnotationView(withIdentifier: identifier)
        if annotationView == nil {
            annotationView = MKMarkerAnnotationView(annotation: annotation, reuseIdentifier: identifier)
            annotationView?.canShowCallout = true
            annotationView?.rightCalloutAccessoryView = UIButton(type: .detailDisclosure)
        } else {
            annotationView?.annotation = annotation
        }

        return annotationView
    }
    
    func mapView(_ mapView: MKMapView, annotationView view: MKAnnotationView, calloutAccessoryControlTapped control: UIControl) {
        guard let placemark = view.annotation as? MKPointAnnotation else { return }
        parent.selectedPlace = placemark
        parent.showingPlaceDetails = true
    }
}

struct MyBodyView: View {
    @State private var centerCoordinate = CLLocationCoordinate2D()
    @State private var locations = [MKPointAnnotation]()
    @State private var selectedPlace: MKPointAnnotation?
    @State private var showingPlaceDetails = false
    
    var body: some View {
        ZStack {
            MyMapView(centerCoordinate: $centerCoordinate, annotations: locations, selectedPlace: $selectedPlace, showingPlaceDetails: $showingPlaceDetails)
                .edgesIgnoringSafeArea(.all)
            Spacer()
            HStack {
                Spacer()
                Button(action: {
                    let newLocation = MKPointAnnotation()
                    newLocation.coordinate = self.centerCoordinate
                    newLocation.title = "Example location"
                    self.locations.append(newLocation)
                }){
                    Image(systemName: "plus")
                }
                .padding()
                .background(Color.black.opacity(0.75))
                .foregroundColor(.white)
                .font(.title)
                .clipShape(/*@START_MENU_TOKEN@*/Circle()/*@END_MENU_TOKEN@*/)
                .padding(.trailing)
            }
            Circle()
                .fill(Color.blue)
                .opacity(0.3)
                .frame(width: 32, height: 32)
        }
        .alert(isPresented: $showingPlaceDetails) {
            Alert(title: Text(selectedPlace?.title ?? "Unknown"),
                  message: Text(selectedPlace?.subtitle ?? "Missing place infomation."),
                  primaryButton: .default(Text("OK")),
                  secondaryButton: .default(Text("Edit")) {
                
            })
        }
    }
}
#Preview {
    MyBodyView()
}

```
![image](/assets/img/swiftui/swiftui_map1.png)
![image](/assets/img/swiftui/swiftui_map2.png)
![image](/assets/img/swiftui/swiftui_map3.png)